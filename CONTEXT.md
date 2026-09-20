# Home Server

The whole system of Hosts, Guests, and Services run at home for the Operator and Household Consumers. This repo is the source of truth later agents use to install and change that system.

## Language

**Home Server**:
The whole system: Hosts, Guests, and Services together. Not a single machine.
_Avoid_: the laptop, the box, the server (ambiguous)

**Host**:
A physical machine that runs Proxmox. The Laptop Host is Host 1 and stays. The Desktop Host is a future Host 2 for heavier work and is not specified here.
_Avoid_: node, server, box, hypervisor (the product, not the machine)

**Guest**:
A virtual machine or container that runs on a Host.
_Avoid_: VM (too narrow), container (too narrow), box

**Service**:
An application people actually use, such as the Vault, DNS Filter, or Media Server.
_Avoid_: app, container, stack (a stack is several Services)

**Laptop Host**:
Host 1. A small 2-in-1 laptop kept permanently for light Services. Not a trial box and not a future cluster partner. It has no built-in Ethernet jack; the LAN Uplink is a USB Ethernet adapter.
_Avoid_: the home server, the Proxmox box

**Uplink**:
The wired Ethernet path from the Laptop Host to the home LAN. On this Host it is a USB-C adapter, not a built-in jack. If it falls out, the Home Server is off the network.
_Avoid_: NIC, ethernet, dongle (the object, not the role)

**Gateway**:
The ISP modem/router that currently hands out LAN addresses and Wi-Fi. Today that is a Cox Panoramic Wifi CGM4140COM. It is not a Host, and we do not fully control its DNS.
_Avoid_: router (ambiguous once a second router exists), modem, combo box

**Desktop Host**:
A future second Host for heavier work. Named only so later agents do not treat the Laptop Host as the final machine. No spec lives here.
_Avoid_: the cluster, the main server, the real server

**Operator**:
The person who administers the Home Server and is the only one with remote Tailscale access.
_Avoid_: admin, user, owner, you

**Operator Station**:
The Operator’s daily computer. It is not a Host. Today that is an Acer Nitro V15. Backups and admin work happen here, not on the Laptop Host.
_Avoid_: my laptop, the other laptop, the main PC

**Household Consumer**:
A person on the home LAN who may use the Media Server, with no Tailscale account and no admin access. They are not pointed at the DNS Filter in this phase.
_Avoid_: user, family member, client, guest (collides with Guest)

**Vault**:
The password-manager Service, used only by the Operator.
_Avoid_: Bitwarden (a product), password manager (generic)

**DNS Filter**:
The Service that answers DNS on the LAN and blocks ads and trackers for devices that use it. In this phase only Operator devices are pointed at it.
_Avoid_: ad blocker (often means a browser extension), Pi-hole (a product)

**Media Server**:
The Service that plays files from the Media Library to a screen. It is not the downloader or organizer.
_Avoid_: media manager, Plex (a product), Jellyfin (a product)

**Media Library**:
The canonical video and audio files, kept on always-on external disks, not on the Host boot disk. Distinct from the Media Server that plays them and from any Library Stack that fetches them.
_Avoid_: media, downloads, collection (vague)

**Library Disk**:
The USB disk mounted on the Laptop Host that holds the Media Library. The first Library Disk is a temporary 2 TB Passport, not the long-term disk.
_Avoid_: the hard drive, external, USB (the bus, not the role)

**Library Stack**:
The optional Services that search for, download, and file media into the Media Library (Sonarr, Radarr, and friends). Not part of the first install; only after the Vault, DNS Filter, and Media Server already work.
_Avoid_: media manager, *arr (jargon without the split from Media Server)

**DNS Guest**:
The Guest that runs only the DNS Filter, so experiments elsewhere cannot take down DNS for anyone using it.
_Avoid_: the Pi-hole VM, the adblock container

**App Guest**:
The Guest that runs the Vault and the Media Server.
_Avoid_: the Docker VM, the media box

**Light Service**:
A Service the Laptop Host is meant to keep: Vault, DNS Filter, and a playback-oriented Media Server.
_Avoid_: small service, easy service

**Heavy Service**:
A Service reserved for the Desktop Host, especially work that needs sustained transcode or many concurrent streams. Compute-light but operationally heavy work may still run on the Laptop Host if we choose that later.
_Avoid_: big service, media (ambiguous)
