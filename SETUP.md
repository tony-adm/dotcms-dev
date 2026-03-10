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
```

---

## 5. Point DNS

Set A records for both `developer.black` and `app.developer.black` to `154.53.45.186`.

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

## Pending / TODO

- [ ] Add tawk.to live chat widget to site
- [ ] Cold email prospecting for law firms / slow websites
- [ ] VTL templates for dynamic content rendering
