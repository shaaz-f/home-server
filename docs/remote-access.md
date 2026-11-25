# Remote Access & Management

[← Back to overview](../README.md)

## WireGuard VPN

| | |
|---|---|
| Server | WireGuard Server 1 on the UCG Ultra |
| Listen port | UDP 51820 |
| Clients | 2 |
| Access once connected | Internal, Untrusted, Internet |

## NetBird VPN (home server)

| | |
|---|---|
| Server | Self-hosted on the `network` VM at `vpn.example.ca` (management, signal, relay, STUN, dashboard) |
| Sign-in | Pocket ID (`id.example.ca`), passkey; local email login disabled; new users need approval |
| Inbound ports | TCP 443, UDP 3478 (port forwards to `network`) |
| Routing peer | `apps` VM (Docker client, group `routers`, masquerade on) |
| Network `Home` | Resource `apps VM` (10.0.30.3) |
| Access policies (one-way) | `admins` → `res-apps`; `admins` → `routers`, TCP 8888 (test); default allow-all policy removed |

## NetBird VPN (cloud VPS)

Self-hosted on the cloud VPS at `nb.example.ca`, sign-in through Pocket ID at `oidc.example.ca`. See [Cloud VPS](cloud.md).

## UniFi console management

| Setting | State |
|---|---|
| Remote Access (unifi.ui.com) | On |
| Direct Remote Connection | Off |
| SSH | Off |
| Automatic Backups | On (weekly) |
| Analytics | Minimum |
| Firmware | UniFi OS 6.0.17, Network 11.0.99; all devices up to date |
