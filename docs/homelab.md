# Homelab

[← Back to overview](../README.md)

## Proxmox VE host

| | |
|---|---|
| Connection | USW Lite 8 PoE, port 5 (trunk) |
| Host management network | Management (VLAN 10) |
| VM networks | Home (20), Private (30), Public (40) via VLAN tags on the Proxmox bridge |

```mermaid
flowchart TD
    SW["USW Lite 8 PoE · port 5 (trunk)"] --> BR["Proxmox bridge (VLAN-aware)"]
    BR -- "untagged · VLAN 10" --> HOST["Proxmox host UI"]
    BR -- "tag 30" --> CADDY["management (Caddy, no Docker)"]
    BR -- "tag 30" --> APPS["apps (Docker)"]
    BR -- "tag 30" --> MON["monitoring (Docker)"]
    BR -- "tag 40" --> NET["network (Caddy, Pocket ID, NetBird, TinyAuth)"]
    BR -- "tag 20" --> HERMES["hermesagent (stopped)"]
```

## Guests

| Name | Type | Network | IP | Resources |
|---|---|---|---|---|
| `management` | LXC (Debian 13, unprivileged, no Docker) | Private (30) | 10.0.30.4 (fixed) | 1 vCPU, 1 GB RAM, 8 GB disk |
| `apps` | VM (Debian 13, Docker) | Private (30) | 10.0.30.3 (fixed) | 4 GB RAM, 40 GB disk |
| `monitoring` | VM (Debian 13, Docker) | Private (30) | 10.0.30.2 (fixed) | 2 vCPU, 2 GB RAM, 30 GB disk |
| `network` | VM (Debian 13, Docker) | Public (40, DMZ) | 10.0.40.2 (fixed) | – |
| hermesagent | LXC (stopped) | Home (20) | DHCP | 4 GB RAM |

## Services

| Name | Host | Port | Role |
|---|---|---|---|
| Caddy (custom build with the Cloudflare DNS module, systemd service) | `management` | TCP 80, 443 | Reverse proxy for admin tools; wildcard certificate for `*.example.ca` (Cloudflare DNS challenge) |
| Uptime Kuma | `monitoring` | TCP 3001 | Uptime monitoring; alerts to a private Discord channel. Monitors: Pocket ID (HTTPS), NetBird (HTTPS), Caddy on `network` (TCP 443) |
| Beszel hub | `monitoring` | TCP 8090 | CPU, RAM and disk graphs |
| Komodo Core, Periphery, MongoDB | `monitoring` | TCP 9120 (Core web UI) | Docker management; sign-in through Pocket ID (OIDC), local login kept; web terminals disabled |
| Komodo Periphery | `apps` | – (connects out to Core through `komodo.example.ca`) | Docker agent; terminals and container exec disabled; working directory `/opt/komodo` |
| Beszel agent | Proxmox host, `monitoring`, `apps` | – | Reports host and container metrics to the hub |
| Beszel agent (no Docker socket) | `network` | TCP 45876 (hub connects in) | Reports host metrics to the hub |
| Caddy (custom build with the Cloudflare DNS module) | `network` | TCP 80, 443 (443/udp not forwarded) | Reverse proxy with automatic HTTPS (DNS challenge) |
| Pocket ID v2 | `network` | 1411 (Docker network only) | Identity provider (passkeys) |
| NetBird server and dashboard | `network` | 80 (Docker network only), UDP 3478 | Self-hosted VPN control plane, relay and STUN |
| NetBird client `netbird-nb` (routing peer) | `apps` | – | Routes cloud NetBird traffic to 10.0.30.4; previous client for the home NetBird server stopped |
| TinyAuth v5.2.0 | `network` | 3000 (Docker network only) | Forward-auth gate for Caddy; signs in through Pocket ID (OAuth only, email whitelist) |

The monitoring web UIs are reached by name through Caddy on `management`, or directly by IP and port from Admin Devices.

## Internal DNS

See [Networks](networks.md) for the internal and public DNS records.

## Reverse proxy routes

Caddy on `network` (10.0.40.2), reached through the port forwards:

| Hostname | Backend | Access |
|---|---|---|
| `id.example.ca` | `pocket-id:1411` | Public |
| `vpn.example.ca` | NetBird server and dashboard (`netbird-server:80`, `netbird-dashboard:80`) | Public |
| `auth.example.ca` | `tinyauth:3000` | Public (login page) |

A reusable Caddy snippet `tinyauth` (forward auth to `tinyauth:3000`, endpoint `/api/auth/caddy`) can be imported by any site block to require a Pocket ID login.

Caddy on `management` (10.0.30.4), wildcard certificate for `*.example.ca`:

| Hostname | Backend | Access |
|---|---|---|
| `pm.example.ca` | Proxmox UI, `https://10.0.10.10:8006` | Admin Devices and the WireGuard subnet (192.168.2.0/24); others get 403 |
| `kuma.example.ca` | Uptime Kuma, `http://10.0.30.2:3001` | Admin Devices, the WireGuard subnet and 10.0.30.3 (`apps`); others get 403 |
| `beszel.example.ca` | Beszel hub, `http://10.0.30.2:8090` | Admin Devices, the WireGuard subnet and 10.0.30.3 (`apps`); others get 403 |
| `komodo.example.ca` | Komodo Core, `http://10.0.30.2:9120` | Admin Devices, the WireGuard subnet and 10.0.30.3 (`apps`); others get 403 |
| any other name | – | 404 |
