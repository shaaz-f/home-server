# Firewall & Zones

[← Back to overview](../README.md)

Firewall: UniFi **Zone-Based Firewall**. Policies apply between zones.

## Zone membership

| Zone | Networks / interfaces |
|---|---|
| Internal | Default, Home |
| Mgmt | Management |
| Lab | Private |
| Untrusted | Public, IoT, Guest |
| External | Internet 1 (WAN1), Internet 2 |
| Gateway | The UCG Ultra itself |
| VPN | WireGuard Server 1 |
| Hotspot | – |
| DMZ | – |

## Zone matrix

Rows are the **source** and columns the **destination**. "Return" means only replies to connections started from the other side are allowed.

| Source ↓ / Dest → | Internal | External | Gateway | VPN | Lab | Mgmt | Untrusted |
|---|---|---|---|---|---|---|---|
| **Internal** | ✅ Allow | ✅ Allow | ✅ Allow | ✅ Allow | 🔑 Admin Devices: all · others: TCP 80/443 | 🔑 Admin Devices only | ✅ Allow |
| **External** | ↩️ Return | ↩️ Return | ↩️ Return* | ↩️ Return | ↩️ Return | ↩️ Return | ↩️ Return |
| **Gateway** | ✅ Allow | ✅ Allow | – | ✅ Allow | ✅ Allow | ✅ Allow | ✅ Allow |
| **VPN** | ✅ Allow | ✅ Allow | ✅ Allow | ✅ Allow | ✅ Allow (custom) | ✅ Allow (custom) | ✅ Allow |
| **Lab** | ↩️ Return | ✅ Allow | ✅ Allow | ↩️ Return | – | ⛔ Block (except Caddy → Proxmox TCP 8006) | ⛔ Block |
| **Mgmt** | ↩️ Return | ✅ Allow | ✅ Allow | ↩️ Return | ⛔ Block | – | ⛔ Block |
| **Untrusted** | ↩️ Return | ✅ Allow | ⛔ Block (custom) | ↩️ Return | ⛔ Block | ⛔ Block | ⛔ Block |

\* External → Gateway also allows WireGuard (UDP 51820), IPTV multicast/IGMP, and IPv6 control traffic (built-in).

```mermaid
flowchart LR
    I[Internal] -->|Admin Devices: all<br/>others: TCP 80/443| L[Lab]
    L -->|Caddy → Proxmox TCP 8006| M
    I -->|Admin Devices only| M[Mgmt]
    I -->|allow| U[Untrusted]
    V[VPN] -->|allow| I
    V -->|allow| L
    V -->|allow| M
    L -.->|return only| I
    M -.->|return only| I
    L x--x|block| M
    U -.->|return only| I
    U -->|block| G[Gateway]
    L -->|allow| E[Internet]
    M -->|allow| E
    U -->|allow| E
    E -->|UDP 51820| G
```

## Custom policies

| Name | Action | Source | Destination |
|---|---|---|---|
| Allow All Traffic | Allow | Internal | Untrusted |
| Allow All Traffic | Allow | VPN | Untrusted |
| Block all Traffic | Block | Untrusted (Guest, IoT, Public) | Gateway |
| Allow Admin Devices to Lab | Allow | Internal (Admin Devices) | Lab |
| Allow Internal to Lab Web | Allow, TCP 80, 443 | Internal | Lab |
| Allow Caddy to Proxmox UI | Allow, TCP 8006 | Lab (10.0.30.237) | Mgmt (10.0.10.10) |
| TEMP Allow VPN to Lab | Allow | VPN | Lab |
| Allow Admin Devices to Mgmt | Allow | Internal (Admin Devices: 10.0.20.10, 10.0.20.11) | Mgmt |
| TEMP Allow VPN to Mgmt | Allow | VPN | Mgmt |

Each Allow policy has a matching return-traffic policy created by UniFi.

## Port forwarding

None.

## NAT

Masquerade NAT from all VLANs to Internet 1 and Internet 2 (UniFi defaults).
