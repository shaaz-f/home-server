# Home Network & Homelab

A segmented home network on the Ubiquiti UniFi platform, with seven VLANs, a zone-based firewall and a Proxmox homelab.

> **Last updated:** 2026-11-17 · **UniFi Network:** 11.0.99 · **UniFi OS:** 6.0.17
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
| **Firewall** | UniFi Zone-Based Firewall: Internal / Untrusted / VPN / External |
| **Remote access** | WireGuard VPN |
| **Port forwards** | None |

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
    VMS["VMs / containers<br/>Caddy · Docker · hermesagent"]

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
        MGMT["Management · VLAN 10<br/>10.0.10.0/24"]
        HOME["Home · VLAN 20<br/>10.0.20.0/24"]
        PRIV["Private · VLAN 30<br/>10.0.30.0/24"]
    end
    subgraph UNT["Untrusted zone"]
        PUB["Public · VLAN 40<br/>10.0.40.0/24"]
        IOT["IoT · VLAN 50<br/>10.0.50.0/24"]
        GST["Guest · VLAN 100<br/>10.0.100.0/24"]
    end
    subgraph VPNZ["VPN zone"]
        WG["WireGuard Server 1"]
    end

    INT -- "allow" --> UNT
    UNT -. "return traffic only" .-> INT
    UNT -- "allow" --> WAN
    INT -- "allow" --> WAN
    WG -- "allow" --> INT
    WAN -. "UDP 51820 only" .-> WG
```

## Features

- Seven VLANs across two firewall zones (Internal and Untrusted), plus a VPN zone
- Switch, AP and Proxmox host on a dedicated Management VLAN (10)
- Homelab services (Caddy, Docker) on the Private VLAN (30)
- No inbound port forwards; WireGuard VPN for remote access
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
