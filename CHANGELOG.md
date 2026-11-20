# Changelog

Changes to the network, newest first.

## 2026-11-20
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
