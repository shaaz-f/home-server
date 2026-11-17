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
| **Internal** | ✅ Allow | ✅ Allow | ✅ Allow | ✅ Allow | ✅ Allow (custom) | ✅ Allow (custom) | ✅ Allow |
| **External** | ↩️ Return | ↩️ Return | ↩️ Return* | ↩️ Return | ↩️ Return | ↩️ Return | ↩️ Return |
| **Gateway** | ✅ Allow | ✅ Allow | – | ✅ Allow | ✅ Allow | ✅ Allow | ✅ Allow |
| **VPN** | ✅ Allow | ✅ Allow | ✅ Allow | ✅ Allow | ✅ Allow (custom) | ✅ Allow (custom) | ✅ Allow |
| **Lab** | ↩️ Return | ✅ Allow | ✅ Allow | ↩️ Return | – | ⛔ Block | ⛔ Block |
| **Mgmt** | ↩️ Return | ✅ Allow | ✅ Allow | ↩️ Return | ⛔ Block | – | ⛔ Block |
| **Untrusted** | ↩️ Return | ✅ Allow | ⛔ Block (custom) | ↩️ Return | ⛔ Block | ⛔ Block | ⛔ Block |

\* External → Gateway also allows WireGuard (UDP 51820), IPTV multicast/IGMP, and IPv6 control traffic (built-in).

```mermaid
flowchart LR
    I[Internal] -->|allow| L[Lab]
    I -->|allow| M[Mgmt]
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
| TEMP Allow Internal to Lab | Allow | Internal | Lab |
| TEMP Allow VPN to Lab | Allow | VPN | Lab |
| TEMP Allow Internal to Mgmt | Allow | Internal | Mgmt |
| TEMP Allow VPN to Mgmt | Allow | VPN | Mgmt |

Each Allow policy has a matching return-traffic policy created by UniFi.

## Port forwarding

None.

## NAT

Masquerade NAT from all VLANs to Internet 1 and Internet 2 (UniFi defaults).
