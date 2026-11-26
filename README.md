# Home Network & Homelab

A segmented, zero-inbound home network and Proxmox homelab, built on the Ubiquiti UniFi platform. It has seven VLANs behind a zone-based firewall, passkey single sign-on in front of every self-hosted service, and private remote access through a self-hosted NetBird VPN, without opening a single port at home.

The cloud half of the setup, which hosts the VPN and identity, is in the companion repository **cloud-server**.

> **Last updated:** 2026-11-26

---

## Highlights

- **Seven VLANs, five trust zones.** Infrastructure, homelab services, trusted devices, IoT and guests are each kept apart, and traffic between them is denied unless a policy allows it.
- **Nothing exposed at home.** The home network accepts no inbound connections. Remote access goes through an outbound VPN tunnel to a cloud server.
- **One front door.** Every service sits behind a single reverse proxy with an automatic wildcard certificate, which it gets through a DNS challenge, so the proxy is never reachable from the internet.
- **Passkey single sign-on.** Pocket ID (OIDC, passkeys only) decides who can use what by group. Services without a login of their own get an oauth2-proxy check at the reverse proxy.
- **Least-privilege VPN.** NetBird routes exactly one address, the reverse proxy, and only over HTTPS. A VPN user can reach the services they're allowed and nothing else on the network.
- **Split-horizon DNS.** Service names resolve the same way at home and away, and none of them exist in public DNS.
- **GPU passthrough.** The mini PC's Intel integrated GPU is passed through to a VM for hardware video decoding and encoding.
- **Monitoring and fleet management.** Uptime Kuma, Beszel and Komodo watch and manage every host and container.

---

## Stack

| Layer | Tools |
|---|---|
| Network | UniFi Cloud Gateway, UniFi PoE switch, UniFi WiFi 7 access point |
| Virtualization | Proxmox VE on a mini PC (6-core Intel CPU, 16 GB RAM, integrated GPU) |
| Guests | Debian VMs and LXC containers, Docker Compose |
| Access | Caddy (reverse proxy), oauth2-proxy, Pocket ID (OIDC + passkeys) |
| Remote access | NetBird (self-hosted, WireGuard-based), WireGuard on the gateway as a backup |
| Monitoring | Uptime Kuma, Beszel, Komodo |
| DNS and certificates | Cloudflare (DNS only), Let's Encrypt via DNS challenge |

---

## Network design

### VLANs

| Name | VLAN | Subnet | Zone | Purpose |
|---|---|---|---|---|
| Default | 1 | 192.168.0.0/24 | Internal | Gateway management |
| Management | 10 | 10.0.10.0/24 | Mgmt | Switch, access point, hypervisor |
| Home | 20 | 10.0.20.0/24 | Internal | Personal devices and TV boxes |
| Private | 30 | 10.0.30.0/24 | Lab | Homelab services |
| Public | 40 | 10.0.40.0/24 | DMZ | Reserved for anything public-facing |
| IoT | 50 | 10.0.50.0/24 | Untrusted | Smart devices |
| Guest | 100 | 10.0.100.0/24 | Untrusted | Guest WiFi, with client isolation |

Custom VLANs follow `10.0.<VLAN ID>.0/24`, so an address tells you which network it's on.

### Switching and WiFi

- **The hypervisor** gets a trunk port: Management untagged, and the Home, Private and Public VLANs tagged. Every VM and container is placed on its VLAN by a tag on Proxmox's VLAN-aware bridge.
- **The access point** gets a PoE trunk port: management untagged, and the WiFi VLANs tagged.
- **Two SSIDs**, home and guest. The guest network isolates clients from each other and can only reach the internet.
- **Radio tuning:** a narrow fixed channel on 2.4 GHz, 40 MHz on 5 GHz, DFS off, band steering and BSS transition on.
- **IGMP snooping and an mDNS proxy** keep IPTV and device discovery working across VLANs.

### Zone-based firewall

Rows are the source and columns the destination. "Return" means only replies to connections the other side started.

| Source ↓ / Dest → | Internal | Internet | Gateway | Lab | Mgmt | Untrusted |
|---|---|---|---|---|---|---|
| **Internal** | Allow | Allow | Allow | Admin devices: all · others: web only | Admin devices only | Allow |
| **Lab** | Return | Allow | Allow | – | Hypervisor UI only, for monitoring and the reverse proxy | Block |
| **Mgmt** | Return | Allow | Allow | Block | – | Block |
| **Untrusted** | Return | Allow | **Block** | Block | Block | Block |
| **VPN** | Allow | Allow | Allow | Allow | Allow | Allow |

- **Admin devices** are a short list of fixed-address machines. They're the only clients that can reach the management network or the homelab directly.
- **Everyone else at home** reaches the homelab only through the reverse proxy, over HTTP/HTTPS.
- **Untrusted networks** can't even talk to the gateway, apart from what they need to get online.

---

## Homelab

| Guest | Type | Role |
|---|---|---|
| `management` | Unprivileged LXC, no Docker | Reverse proxy and single sign-on |
| `apps` | VM (q35, UEFI) with Docker and the passed-through GPU | Self-hosted apps, and the NetBird routing peer |
| `monitoring` | VM with Docker | Uptime Kuma, Beszel, Komodo |

**How the guests are built.** Each one is a plain Debian install with:
- root login disabled, and a sudo user who isn't in the `docker` group
- Docker from Docker's own repository, with log rotation
- automatic security updates and the QEMU guest agent
- a fixed address
- automatic start when the host boots.

The pieces that guard access, the reverse proxy and sign-in, run in their own small container with no Docker at all.

### Integrated GPU passthrough

The Intel integrated GPU is handed to the `apps` VM:
1. IOMMU is confirmed on, with the GPU alone in its IOMMU group.
2. The host is kept off the GPU: it's bound to `vfio-pci` at boot and the host's graphics driver is blacklisted.
3. The GPU is attached to the VM as a PCIe device, without its option ROM and not as the primary display. The Proxmox console keeps working.
4. Inside the VM, the Intel firmware and the VA-API driver expose it as a render device for containers.

---

## Identity and access

- **Pocket ID** is the single identity provider, hosted on the cloud server. Sign-in uses **passkeys only**, and **groups** (admins, family) decide access.
- **Apps that support OIDC** sign in against Pocket ID directly, with the allowed groups set per app.
- **Apps without a login of their own, or with too much power** (anything that controls Docker), get an **oauth2-proxy** check at the reverse proxy first. One sign-in covers every service.
- **The reverse proxy (Caddy)** holds one wildcard certificate for the whole domain, issued through a Cloudflare DNS challenge. Names without a configured service get a plain 404.
- **Uptime Kuma** has its own login turned off. A Docker firewall rule only accepts connections to it from the reverse proxy, so the sign-in check can't be bypassed.

---

## Remote access

- **NetBird** is the main VPN. The control plane and identity run on the cloud server; at home, a NetBird client on the `apps` VM is the **routing peer** and connects out to it.
- **Exactly one address is routed:** the reverse proxy, over HTTPS, for the admin group only. NetBird's DNS zone points every service name at it.
- **WireGuard** on the gateway is a separate backup path into the network.

---

## Monitoring and management

| Tool | What it does |
|---|---|
| Uptime Kuma | Checks the cloud services and the reverse proxy, and alerts to a private Discord channel |
| Beszel | CPU, RAM, disk and per-container graphs for the hypervisor and both Docker VMs |
| Komodo | Deploys and manages Docker stacks on both VMs through outbound agents, with OIDC sign-in, and web terminals and container exec disabled |

---

## Hardening at a glance

- No inbound ports open at home.
- SSH and direct remote connections to the gateway are off; the gateway is managed through UniFi's cloud console.
- Weekly automatic gateway backups.
- Separate trust zones, with default-deny between them.
- Passkeys instead of passwords, and group-based access everywhere.
- Unprivileged containers, non-root Docker users, and automatic security updates.
