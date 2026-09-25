# Inventory

Facts later agents should not reinvent. Not a glossary (see `CONTEXT.md`) and not a decision log (see `docs/adr/`).

## Laptop Host

- Model: Dell Inspiron 14 7420 2-in-1
- RAM: 16 GB
- Internal storage: ~500 GB SSD (Proxmox + Guests only)
- CPU: 12 threads (Operator described as 12 cores)
- Ports: 2× USB-C (charger, Uplink), 1× USB-A (Library Disk), HDMI, headset, SD
- No built-in Ethernet
- Uplink: USB-C Ethernet adapter, always plugged in
- Uplink interface: `enxc8a362d64f86` (`vmbr0` `bridge-ports`). Wi-Fi `wlp0s20f3` is not the Uplink. The installer name `nic0` does not exist after boot.
- Library Disk (temporary): 2 TB WD Passport, expendable, USB-A, 24/7 plugged in
- Physical: laptop mode, AC power, lid closed, no sleep on lid close
- Hostname: `host1` (also the Tailscale machine name)

## Gateway

- Cox Panoramic Wifi Gateway, Technicolor CGM4140COM
- Expected: DHCP DNS not Operator-controlled; DHCP cannot be turned off (Cox)

## Operator Station

- Acer Nitro V15 (daily computer, not a Host)
- Off-host backup destination
- Bitwarden cloud export happens here

## Operator access

- Tailscale account exists
- Bitwarden cloud is the current password store (import after Vault is up)
- DNS Filter: Operator devices only (Gateway cannot set DHCP DNS)

## Guests

- DNS Guest hostname: `dns` (AdGuard Home in Debian LXC, no Docker)
- App Guest hostname: `app` (Debian VM, Docker Compose: Vaultwarden + Jellyfin)
- Jellyfin LAN discovery: mDNS (`jellyfin.local` or equivalent)

## LAN addresses

- Subnet: `192.168.0.0/24`
- Gateway: `192.168.0.1`
- First DHCP lease on `vmbr0`: `192.168.0.161` (not the pin)
- Pinned addresses, outside that lease:

| Name | Role | Address |
| --- | --- | --- |
| host1 | Laptop Host | `192.168.0.10/24` |
| dns | DNS Guest | `192.168.0.11/24` |
| app | App Guest | `192.168.0.12/24` |
