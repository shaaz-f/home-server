# Hardware

[← Back to overview](../README.md)

## Network equipment

| Device | Model | Role | Mgmt IP | VLAN |
|---|---|---|---|---|
| Gateway | UniFi Cloud Gateway Ultra (UCG Ultra) | Router, firewall, DHCP, DNS, WireGuard server, UniFi Network controller | 192.168.0.1 | Default (1) |
| Switch | UniFi Switch Lite 8 PoE (USW-Lite-8-PoE) | Managed switch, PoE for the AP (52 W budget) | 10.0.10.218 | Management (10) |
| Access point | UniFi U7 Lite | WiFi 7 access point (2.4 + 5 GHz) | 10.0.10.130 | Management (10) |

## Gateway ports (UCG Ultra)

| Port | Speed | Assignment | Connected to |
|---|---|---|---|
| 1 | GbE | LAN | USW Lite 8 PoE, port 8 (uplink) |
| 2 | GbE | LAN | – |
| 3 | GbE | LAN | – |
| 4 | GbE | LAN | – |
| 5 | 2.5 GbE | **WAN1** | ISP fibre |

## Switch ports (USW Lite 8 PoE)

| Port | PoE | Device | Native network | Notes |
|---|---|---|---|---|
| 1 | PoE+ | U7 Lite AP | Management (10) | Trunk: Home + Guest SSID VLANs |
| 2 | – | Apple TV | Home (20) | |
| 3 | – | Desktop PC | Home (20) | |
| 4 | – | – | | |
| 5 | – | Proxmox host | Management (10) | Trunk: VMs tagged Home (20) / Private (30) / Public (40) |
| 6 | – | IPTV box | Home (20) | IPTV multicast (IGMP) |
| 7 | – | – | | |
| 8 | – | Uplink → UCG Ultra port 1 | | Trunk, all VLANs |

## Endpoints

| Device | Network | Connection | IP |
|---|---|---|---|
| Desktop PC | Home | Switch port 3 | 10.0.20.11 (fixed) |
| MacBook Pro | Home | WiFi (HomeWiFi) | 10.0.20.10 (fixed) |
| iPhone | Home | WiFi (HomeWiFi) | DHCP |
| Apple TV | Home | Switch port 2 | DHCP |
| IPTV box | Home | Switch port 6 | DHCP |
| Proxmox host | Management | Switch port 5 | 10.0.10.10 |
| Caddy (VM/LXC) | Private | via Proxmox | 10.0.30.237 (fixed) |
| `apps` (VM) | Private | via Proxmox | 10.0.30.3 (fixed) |
| `monitoring` (VM) | Private | via Proxmox | 10.0.30.2 (fixed) |
| `network` (VM) | Public | via Proxmox | 10.0.40.2 (fixed) |
| hermesagent (LXC, stopped) | Home | via Proxmox | DHCP |
| WireGuard clients (×2) | VPN | Remote | – |
