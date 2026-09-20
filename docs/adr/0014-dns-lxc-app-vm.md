# DNS Guest is LXC; App Guest is a VM

AdGuard Home runs directly in a Debian LXC (no Docker). Vaultwarden and Jellyfin run in Docker Compose inside a Debian VM. Putting Docker in the DNS Guest, or making everything one Compose stack, would couple household DNS to app experiments. This specializes ADR 0004.