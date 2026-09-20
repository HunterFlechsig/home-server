# Tailscale runs on the Host as a subnet router

Tailscale is installed on the Laptop Host only, advertising the LAN/bridge so the Operator can reach Guests by LAN IP. Installing Tailscale in every Guest duplicates machines and contradicts “Operator is the only Tailscale user.” Funnel stays off (see ADR 0002).
