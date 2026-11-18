# Changelog

Changes to the network, newest first.

## 2026-11-17
- **Firewall:** Internal → Lab limited to Admin Devices (all) and TCP 80/443 for other devices; added Caddy → Proxmox TCP 8006; removed "TEMP Allow Internal to Lab".
- **Homelab:** Caddy fixed IP 10.0.30.237; `pm.example.ca` restricted to Admin Devices in Caddy.
- **Firewall:** Internal → Mgmt limited to the Admin Devices list (policy "Allow Admin Devices to Mgmt"); removed "TEMP Allow Internal to Mgmt".
- **Firewall:** Created zones Lab (Private) and Mgmt (Management); added TEMP allow policies Internal → Lab, VPN → Lab, Internal → Mgmt, VPN → Mgmt.
- **Networks:** Fixed IPs for MacBook (10.0.20.10) and Desktop PC (10.0.20.11); network list "Admin Devices" created.
- **Management:** Automatic Backups enabled (weekly).
- **Docs:** Initial documentation (UniFi Network 11.0.99, UniFi OS 6.0.17).
