# Networks & VLANs

[← Back to overview](../README.md)

## VLANs

| Name | VLAN | Subnet | Gateway | Zone | Used for |
|---|---|---|---|---|---|
| Default | 1 | 192.168.0.0/24 | 192.168.0.1 | Internal | Gateway management address |
| Management | 10 | 10.0.10.0/24 | 10.0.10.1 | Mgmt | Switch, AP, Proxmox host |
| Home | 20 | 10.0.20.0/24 | 10.0.20.1 | Internal | PC, laptop, phone, TVs |
| Private | 30 | 10.0.30.0/24 | 10.0.30.1 | Lab | Homelab services (Caddy, Apps, Monitoring) |
| Public | 40 | 10.0.40.0/24 | 10.0.40.1 | DMZ | `network` VM (Caddy, Pocket ID, NetBird, TinyAuth) |
| IoT | 50 | 10.0.50.0/24 | 10.0.50.1 | Untrusted | No clients |
| Guest | 100 | 10.0.100.0/24 | 10.0.100.1 | Untrusted | Guest WiFi (GuestWiFi) |

Custom VLANs use `10.0.<VLAN ID>.0/24`, with the gateway at `.1`.

## Network-wide settings

| Setting | Value |
|---|---|
| Default security posture | Allow All |
| Gateway mDNS proxy | Custom: all VLANs, all services |
| IGMP snooping | On (Home) |
| IPv6 | Not enabled |

## Network lists

| Name | Type | Values |
|---|---|---|
| Admin Devices | IPv4 | 10.0.20.10, 10.0.20.11 |

## DHCP DNS servers

| Network | DNS servers |
|---|---|
| Public | 1.1.1.1, 9.9.9.9 (manual) |

## Internal DNS

| Record | Type | Target |
|---|---|---|
| `id.example.ca` | A | 10.0.40.2 (`network` VM, Caddy) |
| `vpn.example.ca` | A | 10.0.40.2 (`network` VM, Caddy) |
| `auth.example.ca` | A | 10.0.40.2 (`network` VM, Caddy) |
| `*.example.ca` | A (wildcard), rule disabled | Caddy LXC, 10.0.30.237 (see [Homelab](homelab.md)) |

## Public DNS (Cloudflare)

| Record | Type | Target | Proxy |
|---|---|---|---|
| `home.example.ca` | A | WAN IP, updated by UniFi Dynamic DNS (Cloudflare) | DNS only |
| `id.example.ca` | CNAME | `home.example.ca` | DNS only |
| `vpn.example.ca` | CNAME | `home.example.ca` | DNS only |
