---
description: Coolify setup and remote development workflow for dvlpid.my.id
---

# Coolify Remote Development Workflow

## 📋 Project Context

This repo manages the **Coolify self-hosted PaaS** deployment on the user's **office PC** for remote development workflows.

### Environment Details

| Component | Details |
|-----------|---------|
| **Machine** | Office PC (Windows 11) |
| **Docker** | Docker Desktop (running) |
| **Location** | `c:\coolify` |
| **Domain** | `dvlpid.my.id` (managed by Cloudflare) |
| **GitHub User** | `ihzakarunia` |
| **Remote Tunnel** | Cloudflare Tunnel (configured on this PC) |
| **Current Access** | TeamViewer (looking for alternatives) |

### Container Stack

```yaml
services:
  coolify:        # ghcr.io/coollabsio/coolify:latest - Port 8000
  coolify-redis:  # redis:alpine - Session/queue management
```

### Migration Plan

User has **5+ existing container stacks** to migrate into Coolify for centralized management.

---

## 🚀 Workflow: Code Anywhere → Auto-Deploy

```
┌──────────────────┐
│  Laptop          │  (Windows 11, Git installed)
│  (Anywhere)      │
└────────┬─────────┘
         │ git push
         ▼
┌──────────────────┐
│     GitHub       │  (github.com/ihzakarunia)
└────────┬─────────┘
         │ webhook
         ▼
┌──────────────────┐
│  Office PC       │  (This machine - Coolify)
│  coolify.dvlpid.my.id
└────────┬─────────┘
         ▼
    🌐 app.dvlpid.my.id (LIVE!)
```

---

## 🔧 Initial Setup Steps

### 1. Start Coolify Stack
```powershell
cd c:\coolify
docker compose up -d
```

### 2. Access Coolify Dashboard
- **Local**: http://localhost:8000
- **Remote** (after tunnel): https://coolify.dvlpid.my.id

### 3. Create Cloudflare Tunnel for Coolify
```powershell
# Login to Cloudflare (one-time)
cloudflared tunnel login

# Create tunnel for Coolify
cloudflared tunnel create coolify-tunnel

# Route domain to tunnel
cloudflared tunnel route dns coolify-tunnel coolify.dvlpid.my.id

# Run tunnel (maps coolify.dvlpid.my.id → localhost:8000)
cloudflared tunnel run --url http://localhost:8000 coolify-tunnel
```

### 4. Configure GitHub Integration
1. Open Coolify dashboard
2. Settings → Git Providers → Add GitHub
3. Authorize with `ihzakarunia` account
4. Enable webhook for auto-deploy

### 5. Ensure Auto-Start on PC Restart
- Docker Desktop: Settings → General → ✅ "Start Docker Desktop when you sign in"
- Containers use `restart: unless-stopped` (already configured)

---

## 📱 Remote Access Alternatives (Recommended)

### Option A: Tailscale (Recommended for File Editing)
- **What**: Mesh VPN that connects all your devices privately
- **Pros**: Access office PC like local network, edit files via SMB/network share, RDP access
- **Install**: `winget install Tailscale.Tailscale`
- **Use Case**: Full remote desktop + file access

### Option B: Cloudflare Tunnel + VS Code Remote
- **What**: SSH tunnel via Cloudflare for VS Code Remote-SSH
- **Pros**: Edit files directly in VS Code from laptop
- **Requirement**: Enable SSH on office PC + tunnel SSH port

### Option C: Continue with Coolify Only
- **What**: Use Coolify dashboard for all deployments
- **Pros**: No file editing needed if using Git → GitHub → Coolify workflow
- **Use Case**: Pure GitOps workflow

---

## 📁 Repository Structure

```
c:\coolify\
├── docker-compose.yml      # Coolify stack definition
├── references/
│   └── workflows.txt       # Workflow documentation reference
└── .agent/
    └── workflows/
        └── coolify-setup.md  # This file (AI context)
```

---

## ⚡ Quick Commands

```powershell
# Start Coolify
docker compose up -d

# Check status
docker ps

# View Coolify logs
docker logs coolify -f

# Restart Coolify
docker compose restart

# Stop Coolify
docker compose down
```

---

## 🔗 Important URLs

| Service | URL |
|---------|-----|
| Coolify Dashboard (local) | http://localhost:8000 |
| Coolify Dashboard (remote) | https://coolify.dvlpid.my.id |
| GitHub Profile | https://github.com/ihzakarunia |
| Cloudflare Dashboard | https://dash.cloudflare.com |

---

## 📝 Notes for Future Sessions

- User wants to migrate 5+ existing container stacks to Coolify
- Primary use case: web development (various frameworks)
- Both laptop and office PC have Git + SSH keys configured
- Cloudflare Tunnel already tested on this PC before
- Looking for TeamViewer alternative for remote access
