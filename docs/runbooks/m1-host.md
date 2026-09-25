# Milestone 1 — Host

Wipe the Inspiron 14 7420, install Proxmox on the internal SSD, make it stay awake, put Tailscale on the Host as a subnet router, and pin LAN IPs. Stop when the Operator Station can open the Proxmox UI and `host1` is online in Tailscale with an approved subnet route.

Do not create Guests here. That is [m1-services.md](m1-services.md).

Work from the **Operator Station** (Acer Nitro V15) except where a step says **hands on the 7420**.

## 0. Preconditions

- Nitro has this git repo, a USB stick ≥8 GB, and Tailscale already logged in.
- 7420 charger on USB-C, USB-C Ethernet in the other USB-C, Ethernet cable to the Gateway. Passport stays **unplugged** for this milestone.
- Bitwarden cloud is still the live password store. Export later, after the Vault exists.
- You accept that this wipe destroys Windows on the 7420.

Done when: those are true, not when you feel ready.

## 1. ISO (Nitro)

Download the current **Proxmox VE 9** ISO (x86_64, not ARM) from the official page and verify the SHA256 printed **on that page** (do not trust a hash copied from chat):

https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso

Write it to the USB stick (example on Linux; use Rufus on Windows, DD Image mode):

```bash
lsblk
sudo dd if=proxmox-ve_9.*.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

Replace `sdX` with the USB disk, not a partition (`sdb` not `sdb1`).

Done when: `sha256sum` matches the download page and the USB enumerates as a Proxmox installer.

## 2. Firmware (hands on the 7420)

Boot the USB (F12 boot menu on Dell). In firmware, if the options exist:

- AC power, lid-close does not sleep
- USB always powered when on AC
- Secure Boot can stay on if the installer boots; if it does not, turn Secure Boot off for this machine only

Installer choices:

- Hostname: `host1`
- Filesystem: **ext4** (not ZFS, not btrfs)
- Management NIC: the **USB Ethernet**, not Wi-Fi
- Country/timezone: United States / Los Angeles
- Keyboard: US
- Root password: generate, store in Bitwarden cloud as `proxmox-host1-root`. Never put it in git.
- Email: an address the Operator actually reads (update emails)
- IP: DHCP is fine for the installer; we pin it in step 5

Done when: the installer finishes, the 7420 reboots from the internal SSD (remove the USB), and you get a login on the laptop screen or a DHCP lease on the Gateway.

## 3. Reach the UI (Nitro)

On the Gateway admin page (often `10.0.0.1` or `192.168.0.1` or `10.1.10.1`), find the new device’s IPv4. From the Nitro:

```bash
ssh root@HOST1_DHCP_IP
```

Accept the host key. In a browser on the Nitro: `https://HOST1_DHCP_IP:8006` (TLS warning is expected).

Done when: SSH and the Proxmox UI both work as `root`.

## 4. Stay awake (on host1)

```bash
mkdir -p /etc/systemd/logind.conf.d
cat >/etc/systemd/logind.conf.d/lid.conf <<'EOF'
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
IdleAction=ignore
EOF
systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
systemctl restart systemd-logind
```

Close the lid for two minutes. SSH from the Nitro must stay up.

Done when: lid closed, charger in, SSH still works.

## 5. Stable LAN IP

On host1:

```bash
ip -4 route | awk '/default/ {print $3}'
ip -4 addr show vmbr0
```

That default via is the Gateway. The `vmbr0` CIDR is the LAN (example `10.1.10.0/24`). Pick three addresses **outside** the Gateway’s usual DHCP churn if you can see the pool; otherwise pick high numbers (e.g. `.10` `.11` `.12`) and write them in [docs/inventory.md](../inventory.md):

| Name | Role | Address |
| --- | --- | --- |
| host1 | Laptop Host | |
| dns | DNS Guest | |
| app | App Guest | |

On the Gateway, reserve `host1` if the UI allows it. On host1, set a static address on `vmbr0` in `/etc/network/interfaces` (Proxmox writes this file). Keep `bridge-ports` as the USB Ethernet interface name from `ip link` (often `enx…` or `enp…`, not `wlan0`).

The installer writes `bridge-ports nic0`. That name does not exist after boot. On this Host, `ip link` shows the Uplink as `enxc8a362d64f86` and Wi-Fi as `wlp0s20f3`. If `ifreload -a` prints `bridge port nic0 does not exist`, replace every `nic0` in `/etc/network/interfaces` with `enxc8a362d64f86` (leave address and gateway lines alone), then reload:

```bash
grep -n nic0 /etc/network/interfaces
sed -i 's/nic0/enxc8a362d64f86/g' /etc/network/interfaces
ifreload -a
ip -br link
```

`enxc8a362d64f86` should be UP and show `master vmbr0`. If it is `NO-CARRIER`, reseat the USB-C adapter and the cable to the Gateway. Do not put `wlp0s20f3` in `bridge-ports`.

```bash
ifreload -a
```

Reconnect SSH to the **new** host1 IP. Re-open `https://NEW_IP:8006`.

Done when: inventory has all three IPs, host1 answers on the pinned address after a reboot, and Wi-Fi is not the Uplink (`ip route` default goes out `vmbr0`).

## 6. Updates without a subscription (on host1)

Enterprise repos 401 without a key. Disable them and enable no-subscription. Easiest: UI **host1 → Updates → Repositories** → Add **No-Subscription**, Disable **Enterprise** (and Ceph enterprise if listed).

CLI equivalent (Proxmox VE 9 / Trixie, deb822):

```bash
cat >/etc/apt/sources.list.d/proxmox.sources <<'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

Edit `/etc/apt/sources.list.d/pve-enterprise.sources` and `/etc/apt/sources.list.d/ceph.sources`: add `Enabled: no` to each enterprise stanza. Then:

```bash
apt update
apt full-upgrade -y
reboot
```

Done when: `apt update` has no 401, and `pveversion` shows VE 9.x after reboot.

## 7. Tailscale on the Host (on host1)

```bash
curl -fsSL https://tailscale.com/install.sh | sh
echo 'net.ipv4.ip_forward = 1' >/etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' >>/etc/sysctl.d/99-tailscale.conf
sysctl -p /etc/sysctl.d/99-tailscale.conf
```

Replace `LAN_CIDR` with the subnet from step 5 (example `10.1.10.0/24`):

```bash
tailscale up --hostname=host1 --advertise-routes=LAN_CIDR --accept-dns=false --ssh=false
```

Authenticate in the URL it prints (Operator Tailscale account). In https://login.tailscale.com/admin/machines open `host1` → **Edit route settings** → approve the LAN CIDR. Do not enable Funnel. Do not advertise `0.0.0.0/0` (that would be an exit node).

On the Nitro, Tailscale must **accept subnet routes** (Linux: `tailscale set --accept-routes`). Then:

```bash
tailscale ping host1
```

Done when: `host1` shows Subnets **approved**, Funnel is off, and the Nitro can `tailscale ping host1`.

## Milestone 1 Host is done

All of these are true:

- 7420 runs Proxmox VE 9, hostname `host1`, ext4+LVM, lid closed, no sleep
- Uplink is USB-C Ethernet `enxc8a362d64f86` on `vmbr0`; Wi-Fi `wlp0s20f3` is unused
- Inventory lists IPs for `host1`, `dns`, `app`
- Nitro reaches `https://<host1-ip>:8006` and Tailscale hostname `host1`
- Subnet route for the LAN is advertised and approved
- Passport still unplugged; no Guests yet

Continue at [m1-services.md](m1-services.md).
