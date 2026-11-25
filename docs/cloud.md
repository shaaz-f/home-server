# Cloud VPS

[← Back to overview](../README.md)

## Server

| | |
|---|---|
| Type | Cloud VPS, ARM64, 4 vCPU, 24 GB RAM |
| OS | Ubuntu 24.04 LTS, hostname `net` |
| Disk | 50 GB boot volume |
| Public IP | Reserved (fixed) |
| SSH | Key only; root login off; `AllowUsers ubuntu`; fail2ban (sshd jail) |
| Updates | Unattended security upgrades on |
| Host firewall | Provider image default: inbound SSH only; Docker manages forwarding for published ports |
| Docker | Official repository; log rotation 10 MB × 3 |

## Provider firewall (inbound)

| Source | Protocol | Port | Purpose |
|---|---|---|---|
| Any | TCP | 22 | SSH |
| Any | TCP | 80 | HTTP → HTTPS redirect |
| Any | TCP | 443 | NetBird, Pocket ID |
| Any | UDP | 3478 | NetBird STUN |
| Any | ICMP | type 3 code 4 | Path MTU discovery |

## Services

All in `/opt/stacks/<name>/`.

| Stack | Containers | Hostname | Role |
|---|---|---|---|
| `netbird` | Traefik v3.7.18, NetBird server, NetBird dashboard | `nb.example.ca` | VPN control plane, signal, relay and STUN; Traefik serves HTTPS (Let's Encrypt) for every stack |
| `pocket-id` | Pocket ID v2.19.3 (user 1000, read-only, all capabilities dropped) | `oidc.example.ca` | Identity provider (passkeys); joined to the Traefik network, routed by Docker labels. Groups: `admins`. OIDC clients: `NetBird`, `oauth2-proxy` |

## NetBird (cloud)

| Setting | Value |
|---|---|
| Sign-in | Pocket ID (`oidc.example.ca`) only; local email login disabled (local owner account kept for recovery) |
| Pocket ID client | `NetBird`, allowed group `admins` |
| Group sync | JWT group sync on, claim `groups` |
| New users | Approval required |
| Peer session expiry | 30 days |
| Policies | Default allow-all policy removed; `admins` → `res-admin`, TCP 443 |
| Peers | MacBook, `apps` (routing peer) |

### Networks

| Network | Routing peer | Resource | Group |
|---|---|---|---|
| `Home` | `apps` (group `routers`, masquerade on) | `caddy-admin`, 10.0.30.4/32 | `res-admin` |

### DNS zone

Zone `example.ca`, distributed to `admins`:

| Record | Type | Value |
|---|---|---|
| `*` | A | 10.0.30.4 (`management` Caddy) |
| `nb` | A | Cloud VPS public IP |
| `oidc` | A | Cloud VPS public IP |
