# Homelab

[← Back to overview](../README.md)

## Proxmox VE host

| | |
|---|---|
| Connection | USW Lite 8 PoE, port 5 (trunk) |
| Host management network | Management (VLAN 10) |
| VM networks | Home (20), Private (30) via VLAN tags on the Proxmox bridge |

```mermaid
flowchart TD
    SW["USW Lite 8 PoE · port 5 (trunk)"] --> BR["Proxmox bridge (VLAN-aware)"]
    BR -- "untagged · VLAN 10" --> HOST["Proxmox host UI"]
    BR -- "tag 30" --> CADDY["Caddy (reverse proxy)"]
    BR -- "tag 30" --> APPS["apps (Docker)"]
    BR -- "tag 30" --> MON["monitoring (Docker)"]
    BR -- "tag 20" --> HERMES["hermesagent (stopped)"]
```

## Guests

| Name | Type | Network | IP | Resources |
|---|---|---|---|---|
| Caddy / Pi-hole | LXC | Private (30) | 10.0.30.237 (fixed) | 512 MB RAM |
| `apps` | VM (Debian 13, Docker) | Private (30) | 10.0.30.3 (fixed) | 4 GB RAM, 40 GB disk |
| `monitoring` | VM (Debian 13, Docker) | Private (30) | 10.0.30.2 (fixed) | 2 vCPU, 2 GB RAM, 30 GB disk |
| hermesagent | LXC (stopped) | Home (20) | DHCP | 4 GB RAM |

## Services

| Name | Host | Port | Role |
|---|---|---|---|
| Caddy | Caddy LXC | 80, 443 | Reverse proxy with automatic HTTPS for `*.example.ca` (Cloudflare DNS challenge) |
| Uptime Kuma | `monitoring` | TCP 3001 | Uptime monitoring |
| Beszel hub | `monitoring` | TCP 8090 | CPU, RAM and disk graphs |
| Komodo Core, Periphery, MongoDB | `monitoring` | TCP 9120 (Core web UI) | Docker management; web terminals disabled |
| Beszel agent | Proxmox host, `monitoring`, `apps` | – | Reports host and container metrics to the hub |

The monitoring web UIs are reached directly by IP and port from Admin Devices.

## Internal DNS

Wildcard A record `*.example.ca` on the UCG Ultra → Caddy.

## Reverse proxy routes

| Hostname | Backend | Access |
|---|---|---|
| `pm.example.ca` | Proxmox UI, `https://10.0.10.10:8006` | Admin Devices only (Caddy `remote_ip` match; others get 403) |
| `pi.example.ca` | `http://10.0.30.237:8080` | Any device allowed to reach Lab on TCP 443 |
