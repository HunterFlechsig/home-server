# Agents

This repo is how later agents install and change the Home Server. Use the words in CONTEXT.md. New Services, exposure changes, and ADR reversals need a new ADR plus a human decision.

- Glossary: [CONTEXT.md](CONTEXT.md)
- ADR: [docs/adr/](docs/adr/) — locked decisions
- Inventory: [docs/inventory.md](docs/inventory.md) — hardware, hostnames, IPs
- Runbook m1 Host: [docs/runbooks/m1-host.md](docs/runbooks/m1-host.md) — wipe, Proxmox, Tailscale, lid, stable IPs
- Runbook m1 Services: [docs/runbooks/m1-services.md](docs/runbooks/m1-services.md) — DNS Filter, Vault, Serve, off-host backup
- Runbook m2 Playback: [docs/runbooks/m2-playback.md](docs/runbooks/m2-playback.md) — Jellyfin; only after m1 Services is done
- Runbook m3 Library Stack: [docs/runbooks/m3-library-stack.md](docs/runbooks/m3-library-stack.md) — only after m2 has played a file

Secrets: name them, never invent them, never commit them (ADR 0018).
