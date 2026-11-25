# Remote Access & Management

[← Back to overview](../README.md)

## WireGuard VPN

| | |
|---|---|
| Server | WireGuard Server 1 on the UCG Ultra |
| Listen port | UDP 51820 |
| Clients | 2 (MacBook, iPhone) |
| Client endpoint | A Dynamic DNS hostname (Cloudflare, updated by UniFi) |
| Access once connected | Internal, Untrusted, Internet |

## NetBird VPN

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
