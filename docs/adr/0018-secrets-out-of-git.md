# Secrets stay out of git

Proxmox, Tailscale, Vaultwarden, and Jellyfin secrets are never committed. Through milestone 1 they live in Bitwarden cloud on the Operator Station. After a successful Vault import they live in the Vault. Later agents may name a secret and must stop if the value is missing; they must not invent one or paste one into the repo.
