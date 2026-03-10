# developer.black — Setup Guide

Complete steps to reproduce the current production setup from scratch.

---

## Server

- **Provider:** Contabo VPS
- **OS:** Ubuntu 24.04 LTS
- **IP:** 154.53.45.186
- **Domains:** `developer.black`, `app.developer.black`

---

## 1. Base Server Setup

```bash
apt update && apt upgrade -y
apt install -y curl git ufw fail2ban
```

### SSH Hardening
```bash
# /etc/ssh/sshd_config.d/99-hardened.conf
PasswordAuthentication no
PermitRootLogin prohibit-password
X11Forwarding no
MaxAuthTries 6

systemctl restart ssh
```

---

## 2. Install k3s

```bash
curl -sfL https://get.k3s.io | sh -
# Wait for node to be ready
kubectl get nodes
```

---

## 3. Install cert-manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
# Wait for pods
kubectl wait --namespace cert-manager --for=condition=ready pod --all --timeout=120s
```

### Create ClusterIssuer (Let's Encrypt)
```bash
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-http
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: YOUR_EMAIL@example.com
    privateKeySecretRef:
      name: letsencrypt-http
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

---

## 4. Install ingress-nginx

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml

# Wait for controller to get an external IP
kubectl get svc -n ingress-nginx -w
```

The ingress-nginx LoadBalancer service will bind to the host's public IP automatically on k3s:
- Port 80 → HTTP (also used for Let's Encrypt HTTP-01 challenge)
- Port 443 → HTTPS

---

## 5. Point DNS

Set **A records** at your DNS provider:

| Hostname | Type | Value |
|----------|------|-------|
| `developer.black` | A | `154.53.45.186` |
| `app.developer.black` | A | `154.53.45.186` |

Wait for DNS to propagate before proceeding (use `dig developer.black` to verify).

---

## 6. SSL / TLS — cert-manager + Let's Encrypt

### How it works

```
Browser → developer.black:443
  → ingress-nginx (LoadBalancer on port 443)
    → dotcms Service (port 8080)
      → dotcms Pod (port 8082)

TLS terminated at ingress-nginx using cert stored in secret: dotcms-tls
Certificate issued by: Let's Encrypt via HTTP-01 challenge
Certificate manager: cert-manager ClusterIssuer (letsencrypt-http)
```

### Install cert-manager
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
kubectl wait --namespace cert-manager --for=condition=ready pod --all --timeout=120s
```

### Create ClusterIssuer
```bash
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-http
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: tonycobbmail@gmail.com
    privateKeySecretRef:
      name: letsencrypt-http-account-key
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

Verify it registered successfully:
```bash
kubectl describe clusterissuer letsencrypt-http
# Should show: "The ACME account was registered with the ACME server"
```

### How the certificate is issued
When the Ingress is created with annotation `cert-manager.io/cluster-issuer: letsencrypt-http`:
1. cert-manager creates a `Certificate` resource for `developer.black` and `app.developer.black`
2. Let's Encrypt sends an HTTP-01 challenge to `http://developer.black/.well-known/acme-challenge/...`
3. cert-manager answers via a temporary ingress rule
4. On success, the TLS cert is stored in the `dotcms-tls` secret in the `dotcms` namespace
5. ingress-nginx picks it up and serves HTTPS automatically

### Verify certificate
```bash
kubectl get certificate -n dotcms
kubectl describe certificate dotcms-tls -n dotcms
# Should show: "Certificate is up to date and has not expired"

# Check the secret exists
kubectl get secret dotcms-tls -n dotcms
```

---

## 6. Deploy dotCMS Stack via Helm

### Install Helm (if not present)
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Deploy
```bash
cd /home/tony/.openclaw/workspace/dotcms-dev/helm

# Edit values.yaml — change passwords before deploying!
# postgres.password, dotcms.adminPassword

helm install dotcms ./dotcms --namespace dotcms --create-namespace
```

### Verify
```bash
kubectl get pods -n dotcms
kubectl get ingress -n dotcms
```

dotCMS takes ~5-10 minutes to initialize on first run (starter pack download + DB setup).

---

## 7. Stack Components

| Component | Image | Port | Storage |
|-----------|-------|------|---------|
| dotCMS | `dotcms/dotcms:latest` | 8082 (http), 8443 (https) | 10Gi (`dotcms-shared-pvc`) |
| PostgreSQL | `pgvector/pgvector:pg18` | 5432 | 10Gi (`dotcms-db-pvc`) |
| OpenSearch | `opensearchproject/opensearch:1` | 9200 | 20Gi (`dotcms-os-pvc`) |

- Ingress routes `developer.black` and `app.developer.black` → dotCMS port 8080
- TLS via cert-manager (Let's Encrypt HTTP-01)
- Glowroot APM enabled (port 4000)
- Starter pack: `20260211` from dotCMS Artifactory

---

## 8. Accessing dotCMS Admin

- URL: `https://developer.black/dotAdmin`
- Default admin: `admin@dotcms.com` / password set in `dotcms.adminPassword` (values.yaml)

---

## 9. Useful Commands

```bash
# Check pod status
kubectl get pods -n dotcms

# View dotCMS logs
kubectl logs -n dotcms -l app=dotcms -f

# Restart dotCMS
kubectl rollout restart deployment/dotcms -n dotcms

# Upgrade chart after changes
helm upgrade dotcms ./dotcms -n dotcms

# Uninstall (WARNING: destroys PVCs too unless you delete separately)
helm uninstall dotcms -n dotcms
```

---

## 10. Repo

GitHub: `tony-adm/dotcms-dev`
Local: `/home/tony/.openclaw/workspace/dotcms-dev/`

Content types, VTL templates, and theme files are in the repo.

---

## 11. Full Network / Routing Map

```
Internet
   │
   ▼
154.53.45.186:80 / :443   (Contabo VPS public IP)
   │
   ▼
k3s node (single node)
   │
   ▼
ingress-nginx LoadBalancer Service
  - 80:30804/TCP   (HTTP — used for ACME challenge + redirect)
  - 443:32293/TCP  (HTTPS)
   │
   ▼
ingress-nginx Controller Pod
  - Terminates TLS using Secret: dotcms/dotcms-tls  (Let's Encrypt cert)
  - Routes by hostname:
      developer.black     → dotcms Service :8080
      app.developer.black → dotcms Service :8080
   │
   ▼
dotcms ClusterIP Service (10.43.78.1:8080)
   │
   ▼
dotcms Pod (:8082)
  │          │           │
  ▼          ▼           ▼
postgres   opensearch  glowroot
(5432)     (9200)      (4000, internal only)

SSL Certificate:
  Issuer:   Let's Encrypt (ACME HTTP-01)
  Manager:  cert-manager ClusterIssuer "letsencrypt-http"
  Secret:   dotcms/dotcms-tls
  Domains:  developer.black, app.developer.black
  Auto-renewal: cert-manager handles this automatically
```

### Ingress resource (annotated)
```yaml
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-http   # triggers auto cert issuance
    nginx.ingress.kubernetes.io/proxy-body-size: 100m  # allows large file uploads to dotCMS
```

---

## Pending / TODO

- [ ] Add tawk.to live chat widget to site
- [ ] Cold email prospecting for law firms / slow websites
- [ ] VTL templates for dynamic content rendering
