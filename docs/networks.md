# Networks & VLANs

[← Back to overview](../README.md)

## VLANs

| Name | VLAN | Subnet | Gateway | Zone | Used for |
|---|---|---|---|---|---|
| Default | 1 | 192.168.0.0/24 | 192.168.0.1 | Internal | Gateway management address |
| Management | 10 | 10.0.10.0/24 | 10.0.10.1 | Mgmt | Switch, AP, Proxmox host |
| Home | 20 | 10.0.20.0/24 | 10.0.20.1 | Internal | PC, laptop, phone, TVs |
| Private | 30 | 10.0.30.0/24 | 10.0.30.1 | Lab | Homelab services (management, Apps, Monitoring) |
| Public | 40 | 10.0.40.0/24 | 10.0.40.1 | DMZ | `network` VM (stopped) |
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
| `nb.example.ca` | A | Cloud VPS public IP |
| `oidc.example.ca` | A | Cloud VPS public IP |
| `*.example.ca` | A (wildcard) | 10.0.30.4 (`management` LXC, Caddy; see [Homelab](homelab.md)) |

## Public DNS (Cloudflare)

| Record | Type | Target | Proxy |
|---|---|---|---|
| Dynamic DNS hostname (not listed) | A | WAN IP, updated by UniFi Dynamic DNS | DNS only |
| `nb.example.ca` | A | Cloud VPS public IP | DNS only |
| `oidc.example.ca` | A | Cloud VPS public IP | DNS only |
