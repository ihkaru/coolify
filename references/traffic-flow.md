# Traffic Flow Diagram

## Overview

This document explains how traffic flows from clients to your applications hosted on this setup.

---

## Coolify Apps (*.dvlpid.my.id via Traefik)

```mermaid
sequenceDiagram
    participant Client as 🌐 Client Browser
    participant CF as ☁️ Cloudflare Edge
    participant Tunnel as 🔒 cloudflared<br/>(Docker Container)
    participant Traefik as 🔀 coolify-proxy<br/>(Traefik)
    participant App as 📦 Your App<br/>(Docker Container)

    Client->>CF: HTTPS request to<br/>test.dvlpid.my.id
    Note over CF: SSL Termination<br/>DNS: *.dvlpid.my.id
    CF->>Tunnel: Forward request<br/>(QUIC Protocol)
    Note over Tunnel: Route #4: *.dvlpid.my.id<br/>→ https://coolify-proxy:443<br/>(No TLS Verify)
    Tunnel->>Traefik: HTTPS to port 443
    Note over Traefik: Match Host header<br/>Route to container
    Traefik->>App: HTTP to app<br/>(Docker network)
    App-->>Traefik: Response
    Traefik-->>Tunnel: Response
    Tunnel-->>CF: Response
    CF-->>Client: HTTPS Response
```

---

## Direct Host Apps (coolify, ai, pc1)

```mermaid
sequenceDiagram
    participant Client as 🌐 Client Browser
    participant CF as ☁️ Cloudflare Edge
    participant Tunnel as 🔒 cloudflared<br/>(Docker Container)
    participant Host as 🖥️ Windows Host<br/>(host.docker.internal)
    participant Service as 🤖 Service<br/>(Port 8000/8045/3012)

    Client->>CF: HTTPS request to<br/>coolify.dvlpid.my.id
    Note over CF: SSL Termination
    CF->>Tunnel: Forward request<br/>(QUIC Protocol)
    Note over Tunnel: Route #2: coolify.dvlpid.my.id<br/>→ http://host.docker.internal:8000
    Tunnel->>Host: HTTP to Windows Host
    Host->>Service: Localhost forwarding
    Service-->>Host: Response
    Host-->>Tunnel: Response
    Tunnel-->>CF: Response
    CF-->>Client: HTTPS Response
```

---

## Architecture Overview

```mermaid
flowchart TB
    subgraph Internet
        Client[🌐 Client Browser]
        CF[☁️ Cloudflare Edge<br/>SSL Termination]
    end

    subgraph Docker["🐳 Docker Network (coolify)"]
        Tunnel[🔒 cloudflared<br/>Token-based Tunnel]
        Traefik[🔀 coolify-proxy<br/>Traefik :443]
        
        subgraph CoolifyApps["Coolify Managed Apps"]
            App1[📦 test.dvlpid.my.id]
            App2[📦 engine.dvlpid.my.id]
            App3[📦 any-new-app.dvlpid.my.id]
        end
        
        Realtime[📡 coolify-realtime:6001]
    end

    subgraph WindowsHost["🖥️ Windows Host (host.docker.internal)"]
        Coolify[⚙️ Coolify Dashboard :8000]
        AI[🤖 AI Service :8045]
        PC1[🔧 PC1 Service :3012]
    end

    Client -->|HTTPS| CF
    CF -->|QUIC Tunnel| Tunnel
    
    Tunnel -->|"#4: *.dvlpid.my.id<br/>https://coolify-proxy:443"| Traefik
    Tunnel -->|"#2: coolify.dvlpid.my.id<br/>http://host.docker.internal:8000"| Coolify
    Tunnel -->|"#1: ai.dvlpid.my.id<br/>http://host.docker.internal:8045"| AI
    Tunnel -->|"#3: pc1.dvlpid.my.id<br/>http://host.docker.internal:3012"| PC1
    
    Traefik --> App1
    Traefik --> App2
    Traefik --> App3
    Traefik --> Realtime
```

---

## Route Configuration (Cloudflare Tunnel Dashboard)

| Priority | Subdomain | Service | Notes |
|----------|-----------|---------|-------|
| 1 | `ai.dvlpid.my.id` | `http://host.docker.internal:8045` | AI Service on Windows |
| 2 | `coolify.dvlpid.my.id` | `http://host.docker.internal:8000` | Coolify Dashboard |
| 3 | `pc1.dvlpid.my.id` | `http://host.docker.internal:3012` | PC1 Service |
| 4 | `*.dvlpid.my.id` | `https://coolify-proxy:443` | **Wildcard (LAST!)** - All Coolify apps |

⚠️ **Important:** The wildcard route **MUST be last** so specific routes match first!

---

## Key Points

1. **Cloudflare handles SSL** - All public traffic is HTTPS
2. **Tunnel runs in Docker** - Stable networking via `docker-compose.tunnel.yml`
3. **Route order matters** - Specific routes BEFORE wildcard
4. **host.docker.internal** - Special DNS for reaching Windows host from containers
5. **Traefik routes by hostname** - Automatically routes to correct Coolify app container
6. **No TLS Verify** - Required for wildcard route (self-signed cert on Traefik)

---

## Files

| File | Purpose |
|------|---------|
| `c:\coolify\docker-compose.tunnel.yml` | Cloudflared tunnel container |
| `c:\coolify\docker-compose.yml` | Main Coolify services |
| Cloudflare Dashboard | Route configuration (dashboard-managed) |
