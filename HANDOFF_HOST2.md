# Mission Briefing: Static-to-dotCMS Migration (PowerAdmin Theme)

## 🎯 Objective
Convert the static "PowerAdmin" HTML template into a fully functional **dotCMS** site. 

## 🏗️ Architecture & Constraints
We are operating across two machines due to hardware limitations on the primary server.

*   **Host 1 (Gateway/Server):**
    *   Runs the **dotCMS** instance via Docker (`http://192.168.50.63:8080`).
    *   Holds the **Git Workspace** (`~/dotcms-dev`) with all VTL templates and CSS/JS assets.
    *   **Limitation:** CPU lacks AVX2 instructions, meaning the `dotcli` tool **crashes instantly** here. It cannot perform remote administration tasks.
    *   **Bridge:** Maintains an SSH Reverse Tunnel listening on port `18789` that forwards traffic to the dotCMS container's port `8080`.

*   **Host 2 (Helper Agent - YOU):**
    *   Runs on **Windows/WSL2** (Supports AVX2).
    *   **Role:** The **Executor**. You must run all `dotcli` commands to configure the system, as Host 1 cannot.
    *   **Connection:** You access dotCMS via `http://127.0.0.1:18789` (The Tunnel).

---

## 📋 Task List for Host 2 Agent

You are responsible for the **Structural Setup** of the CMS. Host 1 handles the files; you handle the data model.

### 1. Install the dotCMS CLI (`dotcli`)
Since you are the only machine capable of running this tool, you must install it in your WSL environment.

```bash
# Download and install dotCLI for Linux (WSL)
curl -L https://github.com/dotCMS/dotcli/releases/download/v1.0.0/dotcli_Linux_x86_64.tar.gz -o dotcli.tar.gz
tar -xzf dotcli.tar.gz
sudo mv dotcli /usr/local/bin/
rm dotcli.tar.gz

# Verify installation (Must not crash)
dotcli --version
```

### 2. Connect to the Core via Tunnel
You must connect to the dotCMS instance running on Host 1. Do not use the LAN IP; use the **SSH Tunnel** on localhost port `18789` to ensure stability and bypass firewall rules.

```bash
# Configure the CLI profile
dotcli config set remote.url "http://127.0.0.1:18789"
dotcli config set remote.user "admin@dotcms.com"
dotcli config set remote.password "admin" 

# 🚨 CRITICAL CHECK 🚨
# Run this to confirm you can see the server.
# If this fails, the SSH tunnel on Host 1 is down.
dotcli systems info
```

---

## 📦 Content Type Definitions

You need to create the following Content Types to support the dynamic "PowerAdmin" landing page.

### 🔹 1. Hero Banner
*The main visual at the top of the homepage.*

**File:** `hero-banner.json`
```json
{
  "contentType": "HeroBanner",
  "name": "Hero Banner",
  "description": "Homepage Hero Banner component with background image and CTA",
  "host": "SYSTEM_HOST",
  "fields": [
    {
      "dataType": "text",
      "fieldName": "title",
      "name": "Title",
      "required": true,
      "indexed": true,
      "listed": true
    },
    {
      "dataType": "text",
      "fieldName": "subtitle",
      "name": "Subtitle",
      "required": false
    },
    {
      "dataType": "binary",
      "fieldName": "image",
      "name": "Background Image",
      "required": true
    },
    {
      "dataType": "text",
      "fieldName": "buttonText",
      "name": "Button Text",
      "required": false
    },
    {
      "dataType": "text",
      "fieldName": "buttonUrl",
      "name": "Button URL",
      "required": false
    }
  ]
}
```

### 🔹 2. Feature Item
*Used for the "Features" grid section (icon + title + text).*

**File:** `feature-item.json`
```json
{
  "contentType": "FeatureItem",
  "name": "Feature Item",
  "description": "A single feature item for the features grid",
  "host": "SYSTEM_HOST",
  "fields": [
    {
      "dataType": "text",
      "fieldName": "title",
      "name": "Title",
      "required": true,
      "indexed": true,
      "listed": true
    },
    {
      "dataType": "text",
      "fieldName": "iconClass",
      "name": "Icon Class",
      "required": true,
      "hint": "FontAwesome class (e.g., 'fa-cogs')"
    },
    {
      "dataType": "wysiwyg",
      "fieldName": "description",
      "name": "Description",
      "required": true
    },
    {
      "dataType": "text",
      "fieldName": "linkUrl",
      "name": "Link URL",
      "required": false
    }
  ]
}
```

### 🔹 3. Team Member
*Used for the "Meet the Team" section.*

**File:** `team-member.json`
```json
{
  "contentType": "TeamMember",
  "name": "Team Member",
  "description": "Profile for a team member",
  "host": "SYSTEM_HOST",
  "fields": [
    {
      "dataType": "text",
      "fieldName": "fullName",
      "name": "Full Name",
      "required": true,
      "indexed": true,
      "listed": true
    },
    {
      "dataType": "text",
      "fieldName": "role",
      "name": "Job Title",
      "required": true
    },
    {
      "dataType": "binary",
      "fieldName": "photo",
      "name": "Photo",
      "required": true
    },
    {
      "dataType": "text",
      "fieldName": "bio",
      "name": "Short Bio",
      "required": false
    },
    {
      "dataType": "text",
      "fieldName": "socialTwitter",
      "name": "Twitter URL",
      "required": false
    },
    {
      "dataType": "text",
      "fieldName": "socialLinkedin",
      "name": "LinkedIn URL",
      "required": false
    }
  ]
}
```

### 🔹 4. Testimonial
*Used for customer quotes/reviews.*

**File:** `testimonial.json`
```json
{
  "contentType": "Testimonial",
  "name": "Testimonial",
  "description": "Customer quote or review",
  "host": "SYSTEM_HOST",
  "fields": [
    {
      "dataType": "textarea",
      "fieldName": "quote",
      "name": "Quote",
      "required": true,
      "indexed": true
    },
    {
      "dataType": "text",
      "fieldName": "authorName",
      "name": "Author Name",
      "required": true,
      "listed": true
    },
    {
      "dataType": "text",
      "fieldName": "authorRole",
      "name": "Author Role/Company",
      "required": false
    },
    {
      "dataType": "binary",
      "fieldName": "avatar",
      "name": "Author Avatar",
      "required": false
    }
  ]
}
```

### 🔹 5. Call to Action (CTA)
*A generic full-width CTA block.*

**File:** `call-to-action.json`
```json
{
  "contentType": "CallToAction",
  "name": "Call To Action",
  "description": "Full width CTA block",
  "host": "SYSTEM_HOST",
  "fields": [
    {
      "dataType": "text",
      "fieldName": "headline",
      "name": "Headline",
      "required": true,
      "indexed": true,
      "listed": true
    },
    {
      "dataType": "text",
      "fieldName": "subtext",
      "name": "Subtext",
      "required": false
    },
    {
      "dataType": "text",
      "fieldName": "buttonText",
      "name": "Button Text",
      "required": true
    },
    {
      "dataType": "text",
      "fieldName": "buttonUrl",
      "name": "Button URL",
      "required": true
    }
  ]
}
```

---

## 🚀 Execution Steps

Run these commands in order on Host 2 to apply the schema.

```bash
# 1. Create Definitions
# (Copy-paste the JSON blocks above into local files)

# 2. Push Content Types
dotcli content-types import -f hero-banner.json
dotcli content-types import -f feature-item.json
dotcli content-types import -f team-member.json
dotcli content-types import -f testimonial.json
dotcli content-types import -f call-to-action.json

# 3. Verify
echo "All content types pushed. Checking system..."
dotcli content-types list
```
