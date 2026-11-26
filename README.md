# Home Network & Homelab

A segmented home network on the Ubiquiti UniFi platform, with seven VLANs, a zone-based firewall and a small Proxmox homelab. Self-hosted services sit behind one reverse proxy with single sign-on, and remote access runs through a self-hosted NetBird VPN on a cloud server, documented in the companion repository **cloud-server**.

> **Last updated:** 2026-11-26

**Contents:** [Overview](#overview) · [Hardware](#hardware) · [Networks](#networks--vlans) · [WiFi](#wifi) · [Firewall](#firewall) · [DNS](#dns) · [Homelab](#homelab) · [Services](#services) · [Remote access](#remote-access) · [UniFi console](#unifi-console)

---

## Overview

| | |
|---|---|
| **Internet** | Fibre, DHCP on the gateway's 2.5 GbE WAN port |
| **Gateway** | UniFi Cloud Gateway: routing, firewall, DHCP, DNS, WireGuard server |
| **Switching** | UniFi 8-port PoE managed switch |
| **WiFi** | UniFi WiFi 7 access point (2.4 + 5 GHz) |
| **Homelab** | Proxmox VE on a mini PC (6-core Intel CPU with integrated graphics, 16 GB RAM) |
| **VLANs** | 7: Default, Management, Home, Private, Public, IoT, Guest |
| **Firewall** | UniFi Zone-Based Firewall |
| **Infrastructure** | Caddy (reverse proxy), oauth2-proxy and Pocket ID (single sign-on), NetBird (VPN) |
| **Monitoring** | Uptime Kuma, Beszel, Komodo |
| **Remote access** | NetBird (self-hosted on a cloud server), WireGuard on the gateway |

---

## Hardware

| Device | Role | Management IP | VLAN |
|---|---|---|---|
| UniFi Cloud Gateway | Router, firewall, DHCP, DNS, WireGuard server, UniFi Network controller | 192.168.0.1 | Default (1) |
| UniFi 8-port PoE switch | Managed switching, PoE for the access point | 10.0.10.218 | Management (10) |
| UniFi WiFi 7 access point | Home and guest WiFi | 10.0.10.130 | Management (10) |
| Mini PC | Proxmox VE host | 10.0.10.10 | Management (10) |

**How the switch is wired:**
- **Uplink to the gateway:** a trunk port carrying every VLAN.
- **Access point:** a PoE port with Management (10) as its untagged (native) network and the WiFi VLANs tagged.
- **Proxmox host:** a trunk port with Management (10) native, and Home (20), Private (30) and Public (40) tagged.
- **Wired home devices** (desktop PC, streaming box, IPTV box): access ports on Home (20).

**Fixed client addresses.** The desktop PC (10.0.20.11) and the laptop (10.0.20.10) have fixed addresses and make up the **Admin Devices** list the firewall uses. Other clients use DHCP.

---

## Networks & VLANs

| Name | VLAN | Subnet | Gateway | Zone | Used for |
|---|---|---|---|---|---|
| Default | 1 | 192.168.0.0/24 | 192.168.0.1 | Internal | Gateway management address |
| Management | 10 | 10.0.10.0/24 | 10.0.10.1 | Mgmt | Switch, access point, Proxmox host |
| Home | 20 | 10.0.20.0/24 | 10.0.20.1 | Internal | PCs, laptop, phones, TV boxes |
| Private | 30 | 10.0.30.0/24 | 10.0.30.1 | Lab | Homelab guests |
| Public | 40 | 10.0.40.0/24 | 10.0.40.1 | DMZ | Reserved for public-facing services; nothing running |
| IoT | 50 | 10.0.50.0/24 | 10.0.50.1 | Untrusted | No clients |
| Guest | 100 | 10.0.100.0/24 | 10.0.100.1 | Untrusted | Guest WiFi |

Custom VLANs follow the pattern `10.0.<VLAN ID>.0/24`, with the gateway at `.1`.

| Network-wide setting | Value |
|---|---|
| Default security posture | Allow All; the zone policies below do the restricting |
| mDNS proxy | All VLANs, all services |
| IGMP snooping | On for Home, for IPTV multicast |
| IPv6 | Off |
| DHCP DNS for Public (40) | Public resolvers (Cloudflare, Quad9), set manually |

| Network list | Type | Members |
|---|---|---|
| Admin Devices | IPv4 | The desktop PC and laptop addresses |

---

## WiFi

| SSID | Network | Bands | Security | Client isolation |
|---|---|---|---|---|
| HomeWiFi | Home (20) | 2.4 + 5 GHz | WPA2-Personal | Off |
| GuestWiFi | Guest (100) | 2.4 + 5 GHz | WPA2-Personal | On |

| Radio setting | Value |
|---|---|
| Country | Canada |
| 2.4 GHz | 20 MHz, channel 6 |
| 5 GHz | 40 MHz, DFS channels off |
| WiFi speed profile | Conservative |
| Band steering, BSS transition | On |
| Fast roaming (802.11r) | Off |
| Protected Management Frames | Off |

---

## Firewall

UniFi's **Zone-Based Firewall**: each network belongs to a zone, and policies apply between zones.

| Zone | Members |
|---|---|
| Internal | Default, Home |
| Mgmt | Management |
| Lab | Private |
| Untrusted | IoT, Guest |
| DMZ | Public |
| VPN | WireGuard server |
| External | The internet |
| Gateway | The gateway itself |

### Zone matrix

Rows are the source and columns the destination. "Return" means only replies to connections the other side started are allowed.

| Source ↓ / Dest → | Internal | External | Gateway | VPN | Lab | Mgmt | Untrusted | DMZ |
|---|---|---|---|---|---|---|---|---|
| **Internal** | Allow | Allow | Allow | Allow | Admin Devices: all · others: web only | Admin Devices only | Allow | Allow |
| **External** | Return | Return | Return* | Return | Return | Return | Return | Return |
| **Gateway** | Allow | Allow | – | Allow | Allow | Allow | Allow | Allow |
| **VPN** | Allow | Allow | Allow | Allow | Allow | Allow | Allow | Allow |
| **Lab** | Return | Allow | Allow | Return | – | Block, except the Proxmox UI for the reverse proxy and monitoring | Block | Block |
| **Mgmt** | Return | Allow | Allow | Return | Block | – | Block | Block |
| **Untrusted** | Return | Allow | Block | Return | Block | Block | Block | Block |
| **DMZ** | Return | Allow | Return | Return | Block | Block | Block | – |

\* The gateway also accepts WireGuard, IPTV multicast and built-in IPv6 control traffic from the internet.

### Custom policies

| Name | Action | Source | Destination |
|---|---|---|---|
| Allow All Traffic | Allow | Internal | Untrusted |
| Allow All Traffic | Allow | VPN | Untrusted |
| Block all Traffic | Block | Untrusted (Guest, IoT) | Gateway |
| Allow Admin Devices to Lab | Allow | Internal (Admin Devices) | Lab |
| Allow Internal to Lab Web | Allow, web (HTTP/HTTPS) only | Internal | Lab |
| Allow Caddy to Proxmox UI | Allow, Proxmox UI only | Lab (reverse proxy) | Mgmt (Proxmox host) |
| Allow Monitoring to Proxmox | Allow, Proxmox UI only | Lab (monitoring) | Mgmt (Proxmox host) |
| TEMP Allow VPN to Lab | Allow | VPN | Lab |
| Allow Admin Devices to Mgmt | Allow | Internal (Admin Devices) | Mgmt |
| TEMP Allow VPN to Mgmt | Allow | VPN | Mgmt |

UniFi creates a matching return-traffic policy for each Allow policy. All VLANs use UniFi's default masquerade NAT to the internet.

---

## DNS

Self-hosted services use names under one personal domain.

- **At home,** the gateway's DNS has a wildcard record for the domain that points every service name at the reverse proxy. The cloud server's own names have specific records that override the wildcard and point at the cloud server.
- **On NetBird devices,** NetBird's own DNS zone answers for the domain the same way, so the same addresses work at home and away.
- **Public DNS (Cloudflare, DNS only, not proxied)** has records only for the cloud server's names and for a Dynamic DNS name that tracks the home connection's IP. Home service names have no public records.

---

## Homelab

### Proxmox host

The host's own interface is on Management (10). Its network bridge is VLAN-aware, and each guest gets a VLAN tag. The host is managed only through its web UI and SSH.

### Guests

| Name | Type | Network | IP | Resources | Runs |
|---|---|---|---|---|---|
| `management` | LXC, Debian, unprivileged, no Docker | Private (30) | 10.0.30.4 | 1 vCPU, 1 GB RAM, 8 GB disk | Reverse proxy and sign-in (see [Services](#services)) |
| `apps` | VM, Debian, Docker (q35, UEFI) | Private (30) | 10.0.30.3 | 4 GB RAM, 40 GB disk, integrated GPU passed through | Docker apps, NetBird routing peer |
| `monitoring` | VM, Debian, Docker | Private (30) | 10.0.30.2 | 2 vCPU, 2 GB RAM, 30 GB disk | Monitoring stack |

All three start automatically when the host boots.

**How the VMs are built.** Each one is a plain Debian install (no template) with:
- root login disabled, and a normal user with sudo who is not in the `docker` group, so Docker always runs through `sudo`
- Docker from Docker's own apt repository, with log rotation (10 MB × 3 files)
- automatic security updates and the QEMU guest agent
- a fixed IP.

### Integrated GPU passthrough

The CPU's Intel integrated GPU is handed to the `apps` VM for hardware video decoding and encoding (VA-API / Quick Sync). How it's set up:

1. **IOMMU.** Current Proxmox kernels turn it on by default for Intel CPUs, and the GPU sits alone in its IOMMU group, so it can be passed through cleanly.
2. **Keep the host off the GPU.**
   - The `vfio`, `vfio_iommu_type1` and `vfio_pci` modules load at boot.
   - A file in `/etc/modprobe.d/` assigns the GPU's PCI ID to `vfio-pci`, and the host's Intel graphics driver (`i915`) is blacklisted.
   - The initramfs is rebuilt so this applies from the start of boot.
3. **Attach it to the VM.** It's attached as a PCIe device with its option ROM turned off, and not as the primary GPU, so the Proxmox console keeps its own virtual display.
4. **Inside the VM.**
   - Debian's `non-free-firmware` component is enabled.
   - The Intel graphics firmware and the iHD VA-API driver are installed.
   - The GPU appears as a render device under `/dev/dri`.

---

## Services

### Infrastructure

| Service | Where | Role |
|---|---|---|
| Caddy | `management` | Reverse proxy for every self-hosted service, with a wildcard certificate from Let's Encrypt (DNS challenge through Cloudflare, so nothing needs to be reachable from the internet) |
| oauth2-proxy | `management` | Adds single sign-on in front of services that need it, using Pocket ID groups to decide who gets in |
| Pocket ID | Cloud server | Passkey-based identity provider for every service (see **cloud-server**) |
| NetBird routing peer | `apps` | Connects the homelab to the cloud NetBird, so NetBird devices can reach the reverse proxy and nothing else |

Services that support OIDC sign in directly against Pocket ID. Services without their own suitable login get oauth2-proxy's check at the reverse proxy.

### Monitoring stack

| Service | Where | Role |
|---|---|---|
| Uptime Kuma | `monitoring` | Uptime checks for the cloud services and the reverse proxy; alerts to a private Discord channel. Only reachable through the reverse proxy |
| Beszel | Hub on `monitoring`; agents on the Proxmox host, `monitoring` and `apps` | CPU, RAM, disk and container graphs |
| Komodo | Core on `monitoring`; agents on `monitoring` and `apps` | Deploys and manages the Docker stacks. Signs in through Pocket ID; web terminals and container exec are disabled |

---

## Remote access

### NetBird (main)

A self-hosted NetBird, with the server, dashboard and Pocket ID on the cloud server (see **cloud-server**). In short:
- **Signing in:** devices sign in with a Pocket ID passkey, and only an approved admin group may use the VPN.
- **What's reachable:** NetBird routes **only the reverse proxy's address** into the home network, through the routing peer on `apps`, and only for HTTPS.
- **Names:** NetBird's DNS zone makes service names resolve the same way at home and away.

### WireGuard (backup)

| | |
|---|---|
| Server | Built into the gateway |
| Clients | Laptop, phone |
| Endpoint | The Dynamic DNS name |
| Client subnet | 192.168.2.0/24 |

---

## UniFi console

| Setting | State |
|---|---|
| Remote management (unifi.ui.com) | On |
| Direct Remote Connection | Off |
| SSH | Off |
| Automatic backups | On, weekly |
| Analytics | Minimum |
| Updates | All devices kept current |
