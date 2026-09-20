# No public internet exposure

Services are reached on the LAN and, for the Operator only, over Tailscale. No port forwarding, tunnels, or public hostnames. A later agent will see Tailscale and be tempted to enable Funnel or a Cloudflare tunnel; that is out of scope unless this decision is reversed.
