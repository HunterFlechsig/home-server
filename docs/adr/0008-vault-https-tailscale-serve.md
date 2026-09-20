# Vault HTTPS is Tailscale Serve, not a public certificate

Bitwarden clients need HTTPS. Public Let’s Encrypt over HTTP would need an open port, which is forbidden (ADR 0002). The Operator reaches the Vault at a Tailscale HTTPS name; the Host proxies to the App Guest. No domain, no Funnel, no Caddy-for-the-world. Jellyfin stays HTTP on the LAN.
