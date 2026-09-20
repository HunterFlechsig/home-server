# Host and Guests keep stable LAN IPs

Tailscale Serve, AdGuard, and Jellyfin URLs cannot chase DHCP. At install, read the Gateway’s LAN range and pin three addresses (Laptop Host, DNS Guest, App Guest) via Gateway reservation if it exists, otherwise static config on the Host/Guests. Write the chosen numbers into `docs/inventory.md`. Do not leave Proxmox on rolling DHCP.
