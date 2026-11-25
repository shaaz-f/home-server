# Changelog

Changes to the network, newest first.

## 2026-11-24
- **Cloud NetBird:** Network `Home` with routing peer `apps` (masquerade) and resource `caddy-admin` (10.0.30.4/32, group `res-admin`); policy `admins` → `res-admin` TCP 443.
- **Cloud NetBird:** DNS zone for `example.ca` (distributed to `admins`): `*` → 10.0.30.4, `nb` and `oidc` → cloud VPS.
- **Homelab:** NetBird client on `apps` for the home NetBird server stopped; new client `netbird-nb` connected to the cloud NetBird.
- **Cloud VPS:** Reinstalled with Ubuntu 24.04 (ARM64, 50 GB boot volume); SSH key only, root login off, fail2ban; rpcbind disabled; new reserved public IP; Docker installed.
- **Cloud VPS:** Provider firewall inbound: TCP 22, 80, 443 and UDP 3478.
- **Cloud VPS:** NetBird server and dashboard at `nb.example.ca` with the built-in Traefik; sign-in through Pocket ID only; user approval on; peer session expiry 30 days; default policy removed.
- **Cloud VPS:** Pocket ID v2.19.3 at `oidc.example.ca`, behind Traefik.
- **Networks:** Public and internal A records `nb.example.ca` and `oidc.example.ca` → cloud VPS.

## 2026-11-22
- **Homelab:** Komodo sign-in through Pocket ID (OIDC client `Komodo`, restricted to the `admins` group); local login kept.
- **Homelab:** Komodo Periphery agent on `apps`, connecting out to Core through `komodo.example.ca`.
- **Homelab:** `management` Caddy route `komodo.example.ca` → Komodo Core; `kuma.`, `beszel.` and `komodo.` also accept 10.0.30.3 (`apps`).

## 2026-11-21
- **Homelab:** `management` LXC (10.0.30.4, fixed, Private) running Caddy (custom build with the Cloudflare DNS module, no Docker) with a wildcard certificate; routes `pm.example.ca`, `kuma.example.ca` and `beszel.example.ca` for Admin Devices and the WireGuard subnet.
- **Homelab:** Previous Caddy / Pi-hole LXC (10.0.30.237) removed.
- **Firewall:** "Allow Caddy to Proxmox UI" enabled with source 10.0.30.4.
- **Networks:** Internal wildcard record `*.example.ca` enabled, pointing at 10.0.30.4.
- **Homelab:** TinyAuth v5.2.0 on `network` (Pocket ID sign-in only); Caddy site `auth.example.ca` and a reusable `tinyauth` forward-auth snippet.
- **Networks:** Internal DNS record `auth.example.ca` → 10.0.40.2.

## 2026-11-20
- **Homelab:** Beszel agent on `network` (hub connects to TCP 45876; no Docker socket mounted).
- **Homelab:** Uptime Kuma monitors for Pocket ID, NetBird and Caddy on `network`; alerts sent to a private Discord channel.
- **Firewall:** Added "Allow Monitoring to Network Agent" (Lab 10.0.30.2 → DMZ 10.0.40.2, TCP 45876) and "Allow Monitoring to Network HTTPS" (TCP 443).
- **Remote access:** NetBird policies set to one-way (`admins` → `res-apps`; `admins` → `routers`, TCP 8888); default allow-all policy deleted.
- **Firewall:** "Allow Caddy to Proxmox UI" disabled; return traffic confirmed on for "Allow Monitoring to Proxmox".
- **Homelab:** `network` VM (10.0.40.2, fixed, Public/DMZ) running Caddy (Cloudflare DNS challenge), Pocket ID and NetBird.
- **Homelab:** NetBird client on `apps` as routing peer; NetBird network `Home` with resource `apps VM` (10.0.30.3).
- **Remote access:** NetBird VPN at `vpn.example.ca`, signing in through Pocket ID (`id.example.ca`).
- **Firewall:** Added "Allow Apps to NetBird TCP" (Lab 10.0.30.3 → DMZ 10.0.40.2, TCP 443) and "Allow Apps to NetBird STUN" (UDP 3478).
- **Firewall:** Port forwards TCP 443 and UDP 3478 to 10.0.40.2.
- **Networks:** Public DHCP DNS servers set to 1.1.1.1 and 9.9.9.9; internal DNS records `id.example.ca` and `vpn.example.ca` → 10.0.40.2; wildcard `*.example.ca` rule disabled.
- **DNS:** Public records `id.example.ca` and `vpn.example.ca` (CNAME to `home.example.ca`, DNS only).

## 2026-11-18
- **Homelab:** `monitoring` VM (10.0.30.2, fixed, Private) running Uptime Kuma, Beszel hub and Komodo; Beszel agents on the Proxmox host, `monitoring` and `apps`.
- **Homelab:** Docker host VM renamed `apps`, fixed IP 10.0.30.3, disk grown to 40 GB.
- **Homelab:** hermesagent stopped.
- **Firewall:** Added "Allow Monitoring to Proxmox" (Lab 10.0.30.2 → Mgmt 10.0.10.10, TCP 8006).
- **Firewall:** Public network moved from the Untrusted zone to the DMZ zone.

## 2026-11-17
- **Firewall:** Internal → Lab limited to Admin Devices (all) and TCP 80/443 for other devices; added Caddy → Proxmox TCP 8006; removed "TEMP Allow Internal to Lab".
- **Homelab:** Caddy fixed IP 10.0.30.237; `pm.example.ca` restricted to Admin Devices in Caddy.
- **Firewall:** Internal → Mgmt limited to the Admin Devices list (policy "Allow Admin Devices to Mgmt"); removed "TEMP Allow Internal to Mgmt".
- **Firewall:** Created zones Lab (Private) and Mgmt (Management); added TEMP allow policies Internal → Lab, VPN → Lab, Internal → Mgmt, VPN → Mgmt.
- **Networks:** Fixed IPs for MacBook (10.0.20.10) and Desktop PC (10.0.20.11); network list "Admin Devices" created.
- **Management:** Automatic Backups enabled (weekly).
- **Docs:** Initial documentation (UniFi Network 11.0.99, UniFi OS 6.0.17).
