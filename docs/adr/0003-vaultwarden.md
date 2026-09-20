# Vault is Vaultwarden, not official Bitwarden

The Operator uses Bitwarden clients against Vaultwarden. Official Bitwarden self-host is a multi-container stack sized for organizations (~2–4 GB RAM); Bitwarden Lite is still heavier than we need. The trade-off is vendor support vs a single light process. Vaultwarden is unofficial; keep it off the public internet (see ADR 0002).
