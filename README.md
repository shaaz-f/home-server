# Home Network & Homelab

A segmented home network on the Ubiquiti UniFi platform, with seven VLANs, a zone-based firewall and a Proxmox homelab.

> **Last updated:** 2026-11-20 · **UniFi Network:** 11.0.99 · **UniFi OS:** 6.0.17
> Change history: [CHANGELOG.md](CHANGELOG.md)

---

## At a glance

| | |
|---|---|
| **ISP** | Fibre ISP, DHCP on WAN1 (2.5 GbE) |
| **Gateway** | UniFi Cloud Gateway Ultra (UCG Ultra) |
| **Switching** | UniFi Switch Lite 8 PoE |
| **WiFi** | UniFi U7 Lite (WiFi 7, 2.4 + 5 GHz) |
| **Hypervisor** | Proxmox VE |
| **VLANs** | 7 (Default, Management, Home, Private, Public, IoT, Guest) |
| **Firewall** | UniFi Zone-Based Firewall: Internal / Mgmt / Lab / Untrusted / DMZ / VPN / External |
| **Remote access** | WireGuard VPN, NetBird VPN |
| **Port forwards** | TCP 443, UDP 3478 → `network` VM |

## Physical topology

```mermaid
flowchart TD
    ISP(["🌐 ISP (fibre)"])
    GW["UCG Ultra<br/>Gateway / Firewall / DHCP / DNS<br/>192.168.0.1"]
    SW["USW Lite 8 PoE<br/>10.0.10.218"]
    AP["U7 Lite AP<br/>10.0.10.130"]
    PVE["Proxmox VE host"]
    PC["Desktop PC"]
    ATV["Apple TV"]
    OTV["IPTV box"]
    WIFI(("WiFi clients<br/>HomeWiFi / GuestWiFi"))
    VMS["VMs / containers<br/>Caddy · apps · monitoring · network · hermesagent (stopped)"]

    ISP -- "WAN1 · port 5 · 2.5 GbE" --> GW
    GW -- "port 1 ↔ port 8 (uplink trunk)" --> SW
    SW -- "port 1 · PoE" --> AP
    SW -- "port 2" --> ATV
    SW -- "port 3" --> PC
    SW -- "port 5 · trunk" --> PVE
    SW -- "port 6" --> OTV
    AP -.-> WIFI
    PVE --- VMS
```

## Logical layout (VLANs & firewall zones)

```mermaid
flowchart LR
    subgraph EXT["External zone"]
        WAN["Internet (WAN1)"]
    end
    subgraph INT["Internal zone"]
        DEF["Default · VLAN 1<br/>192.168.0.0/24"]
        HOME["Home · VLAN 20<br/>10.0.20.0/24"]
    end
    subgraph MGZ["Mgmt zone"]
        MGMT["Management · VLAN 10<br/>10.0.10.0/24"]
    end
    subgraph LABZ["Lab zone"]
        PRIV["Private · VLAN 30<br/>10.0.30.0/24"]
    end
    subgraph UNT["Untrusted zone"]
        IOT["IoT · VLAN 50<br/>10.0.50.0/24"]
        GST["Guest · VLAN 100<br/>10.0.100.0/24"]
    end
    subgraph DMZZ["DMZ zone"]
        PUB["Public · VLAN 40<br/>10.0.40.0/24<br/>network VM"]
    end
    subgraph VPNZ["VPN zone"]
        WG["WireGuard Server 1"]
    end

    INT -- "Admin Devices only" --> MGZ
    INT -- "Admin Devices: all · others: TCP 80/443" --> LABZ
    LABZ -- "Caddy, Monitoring → Proxmox TCP 8006" --> MGZ
    INT -- "allow" --> UNT
    INT -- "allow" --> DMZZ
    MGZ -. "return traffic only" .-> INT
    LABZ -. "return traffic only" .-> INT
    UNT -. "return traffic only" .-> INT
    DMZZ -. "return traffic only" .-> INT
    UNT -- "allow" --> WAN
    DMZZ -- "allow" --> WAN
    INT -- "allow" --> WAN
    MGZ -- "allow" --> WAN
    LABZ -- "allow" --> WAN
    WG -- "allow" --> INT
    WG -- "allow" --> MGZ
    WG -- "allow" --> LABZ
    LABZ -- "Apps → network TCP 443, UDP 3478" --> DMZZ
    WAN -. "UDP 51820" .-> WG
    WAN -. "TCP 443, UDP 3478" .-> DMZZ
```

## Features

- Seven VLANs across five firewall zones (Internal, Mgmt, Lab, Untrusted, DMZ), plus a VPN zone
- Switch, AP and Proxmox host on a dedicated Management VLAN (10)
- Homelab services (Caddy, apps, monitoring) on the Private VLAN (30)
- Internet-facing services (Caddy, Pocket ID, NetBird) on the `network` VM in the DMZ (Public VLAN 40)
- Inbound port forwards: TCP 443 and UDP 3478 to the `network` VM; WireGuard (UDP 51820) and NetBird VPNs for remote access
- Untrusted zone blocked from Internal, the Gateway and other untrusted VLANs
- Client device isolation on the guest SSID
- SSH and Direct Remote Connection disabled

## Documentation

| Page | Contents |
|---|---|
| [Hardware](docs/hardware.md) | Devices, roles, IPs and port maps |
| [Networks & VLANs](docs/networks.md) | VLANs, subnets, network-wide settings, internal DNS |
| [Firewall & Zones](docs/firewall.md) | Zones, zone matrix, custom policies |
| [WiFi](docs/wifi.md) | SSIDs, security, radio settings |
| [Homelab](docs/homelab.md) | Proxmox host, VMs/containers, internal DNS |
| [Remote Access & Management](docs/remote-access.md) | WireGuard and console management settings |
| [Changelog](CHANGELOG.md) | Dated changes to the network |

---

<sub>Diagrams use [Mermaid](https://mermaid.js.org/) and render on GitHub, in VS Code (with a Mermaid preview extension) and in Obsidian.</sub>
