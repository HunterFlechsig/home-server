# Milestone 1 — Services

DNS Guest (`dns`) with AdGuard Home, App Guest (`app`) with Vaultwarden, Tailscale Serve for Vault HTTPS, first Operator account, Bitwarden cloud import, off-host backup to the Nitro. Stop when a Bitwarden client on the Nitro unlocks the imported Vault over the Serve HTTPS URL and a Guest backup file exists on the Nitro.

Requires [m1-host.md](m1-host.md) complete. Do not install Jellyfin or the Library Stack.

IPs and hostnames come from [docs/inventory.md](../inventory.md). Secrets stay in Bitwarden cloud until the Vault import succeeds (ADR 0018).

On host1, use official `pveam` / `pct` / `qm`. Do not run third-party “helper” installer scripts.

## 1. DNS Guest (on host1)

```bash
pveam update
pveam available --section system | grep debian-13-standard
```

Download the **current** `debian-13-standard_*_amd64.tar.zst` name the command printed:

```bash
pveam download local debian-13-standard_VERSION_amd64.tar.zst
```

Create CT 100. Substitute `DNS_IP/24` and `GATEWAY_IP` from inventory (example `10.1.10.11/24` and `10.1.10.1`):

```bash
pct create 100 local:vztmpl/debian-13-standard_VERSION_amd64.tar.zst \
  --hostname dns \
  --memory 512 --cores 1 --swap 0 \
  --rootfs local-lvm:8 \
  --net0 name=eth0,bridge=vmbr0,ip=DNS_IP/24,gw=GATEWAY_IP \
  --unprivileged 1 \
  --onboot 1 \
  --start 1 \
  --password
```

`--password` prompts; store it as `proxmox-dns-root` in Bitwarden cloud.

If create fails with `unsupported debian version`, `apt full-upgrade -y && reboot` on host1 first (needs `pve-container` current), then retry. Nesting is off: no Docker in this Guest.

```bash
pct exec 100 -- apt update
pct exec 100 -- apt full-upgrade -y
pct exec 100 -- apt install -y curl ca-certificates
```

Free port 53, then install AdGuard Home:

```bash
pct exec 100 -- bash -c 'mkdir -p /etc/systemd/resolved.conf.d
cat >/etc/systemd/resolved.conf.d/adguardhome.conf <<EOF
[Resolve]
DNS=127.0.0.1
DNSStubListener=no
EOF
mv /etc/resolv.conf /etc/resolv.conf.backup 2>/dev/null || true
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
systemctl reload-or-restart systemd-resolved || true'
pct exec 100 -- bash -c 'curl -sS -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v'
```

On the Nitro, open `http://DNS_IP:3000` (first-run wizard) or `http://DNS_IP` if it already moved to 80. Bind DNS to `DNS_IP:53` (this Guest’s LAN IP), not `127.0.0.53`. Upstream: `1.1.1.1` and `1.0.0.1`. Create the admin user; store it as `adguard-admin`. Enable a default blocklist.

On **Operator devices only** (Nitro, phone): set DNS to `DNS_IP`. Leave the Gateway DHCP DNS alone (ADR 0015). If a device still leaks ads, it is using IPv6 DNS — set IPv6 DNS to the Guest if it has an AAAA, or disable IPv6 DNS on that device.

Done when: `dig @DNS_IP example.com` from the Nitro returns an answer, and a known ad hostname is NXDOMAIN or 0.0.0.0 from that same `@DNS_IP`.

## 2. App Guest (on host1)

Download Debian 13 netinst (amd64) onto host1 local storage (ISO). Create VM 101: 8 GB RAM, 4 vCPUs, 32 GB disk (virtio), `vmbr0`, QEMU agent, on-boot. Install Debian 13: hostname `app`, public repos, OpenSSH server, **no** desktop. Static IP `APP_IP/24`, gateway `GATEWAY_IP`, DNS `1.1.1.1` for the install (this Guest is not the DNS Filter client of record). Store `proxmox-app-root` in Bitwarden cloud.

Enable the QEMU guest agent in the guest:

```bash
apt update && apt install -y qemu-guest-agent
systemctl enable --now qemu-guest-agent
```

Install Docker Engine from Docker’s Debian instructions (https://docs.docker.com/engine/install/debian/), including the Compose plugin. Confirm:

```bash
docker run --rm hello-world
docker compose version
```

Done when: `app` pings from host1 at `APP_IP`, SSH as root works from the Nitro, Docker runs `hello-world`.

## 3. Vaultwarden (on app)

Copy `services/app/compose.yml` from this repo onto `app` (scp from the Nitro). Copy `.env.example` to `.env` on `app` (not in git).

Generate an admin token hash **on app**:

```bash
docker run --rm -it vaultwarden/server:latest /vaultwarden hash
```

Store the **password** you typed as `vaultwarden-admin` in Bitwarden cloud. Put the **hash** in `.env` as `ADMIN_TOKEN`. If Compose mangles the hash, double every `$`.

Leave `DOMAIN` as a placeholder until Serve exists. `SIGNUPS_ALLOWED=true`. Then:

```bash
cd /root/app   # or wherever compose.yml lives
docker compose up -d
curl -sI http://127.0.0.1:8080 | head
```

Done when: `curl` from `app` to `:8080` returns HTTP from Vaultwarden (not connection refused).

## 4. Tailscale Serve (on host1)

In https://login.tailscale.com/admin/dns enable **HTTPS Certificates** if it is not already on.

```bash
tailscale serve --bg http://APP_IP:8080
tailscale serve status
```

The status line is the Vault URL (`https://host1.<tailnet>.ts.net`). Put that exact URL (including `https://`) in `app`’s `.env` as `DOMAIN`, then:

```bash
docker compose up -d
```

From the **Nitro with Tailscale up**:

```bash
curl -sI https://host1.TAILNET.ts.net | head
```

If Serve rejects a non-localhost target, on host1 add a loopback forward and Serve that instead:

```bash
apt install -y socat
# systemd unit that runs: socat TCP-LISTEN:8080,bind=127.0.0.1,fork TCP:APP_IP:8080
tailscale serve --bg http://127.0.0.1:8080
```

Do not enable Funnel. Do not open Gateway ports.

Done when: Nitro browser opens the Serve HTTPS URL and shows the Vaultwarden/Bitwarden web vault.

## 5. First account and import (Operator)

1. Create **one** account at the Serve URL (email you actually use).
2. On `app`, set `SIGNUPS_ALLOWED=false` in `.env` and `docker compose up -d`.
3. On the Nitro, Bitwarden cloud web vault → **Export** (encrypted JSON). Store the export on the Nitro, not in git.
4. In the new web vault: **Import**. Unlock in the Bitwarden **browser extension** and the **phone app**, each set to **Self-hosted** with the Serve HTTPS URL.
5. Keep the Bitwarden **cloud** account until that phone and that browser have unlocked the imported Vault (ADR 0010).

Done when: phone and Nitro extension both unlock the self-hosted Vault and show imported logins.

## 6. Off-host backup (host1 → Nitro)

On host1:

```bash
vzdump 100 101 --mode snapshot --compress zstd --storage local --notes-template "m1 {{guestname}}"
ls -lh /var/lib/vz/dump/
```

From the Nitro, pull the dump files into a folder that is **not** this git repo:

```bash
mkdir -p ~/home-server-backups
scp root@HOST1_IP:/var/lib/vz/dump/vzdump-* ~/home-server-backups/
```

Also download an **encrypted** Vault export from the web vault onto the Nitro (same folder).

Done when: Nitro disk has both `vzdump-lxc-100-*.tar.zst` (or equivalent) and `vzdump-qemu-101-*` plus a Vault export file.

## Milestone 1 Services is done

All of these are true:

- `dns` (CT 100) serves AdGuard; Operator devices use it; Gateway DHCP DNS unchanged
- `app` (VM 101) runs Vaultwarden on `:8080`
- Operator reaches the Vault only at the Tailscale Serve HTTPS URL
- Signups are off after the one Operator account
- Cloud import succeeded on phone and Nitro extension; cloud account still exists
- Guest dumps and a Vault export exist on the Nitro
- No Jellyfin, no Library Stack, no public ports, Passport still unused

Next: [m2-playback.md](m2-playback.md).
