# Milestone 2 — Playback

Jellyfin on the App Guest, Library Disk (Passport) mounted on the Host and given to `app`, mDNS so a TV can find it, Operator admin + House user, one real file plays.

Do not start this runbook until [m1-services.md](m1-services.md) is done. Do not install the Library Stack.

This file is a **gate** until you are ready to write the commands. Completion criterion for *opening* the full runbook: milestone 1 Services checks are all true.

When implementing, the runbook must include:

1. Plug the Passport into USB-A, leave it plugged in, format **ext4** (it is expendable), mount by UUID on host1, fstab.
2. Pass that mount into VM 101 (virtiofs or a dedicated SCSI disk). Do not USB-passthrough the whole dongle if the Uplink must stay on USB-C.
3. Add Jellyfin to `services/app/compose.yml` (direct play, no transcode project). HTTP on the LAN. mDNS (`jellyfin.local` or Avahi).
4. Operator admin + House user (ADR 0011).
5. Play one file on a LAN client. That is done. Stop.

Until those commands live here, later agents stop after printing: **m1 is done; m2 runbook is not written yet — do not invent a Jellyfin install.**
