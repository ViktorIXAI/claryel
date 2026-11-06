# P2002 installation log — 2026-11-06

Status: owner-reported observations, not independently verified by remote machine inspection. Public sanitized narrative, no addresses, credentials or serials.

## Installation
- Firmware factory reset; 64 GB USB installer retained in the host for planned recovery.
- Ubuntu Server 24.04 LTS, English, minimal installation; skipped Ubuntu Pro and optional packages; enabled OpenSSH without importing keys.
- Wired DHCP via router; Wi-Fi hardware unused.
- Manual partitioning: planned 400 GB unencrypted ext4 root on approximately 2 TB internal drive; actual layout remains unverified by host commands.
- Installer reported completion and requested reboot. USB left connected.

## Post-install graphics
- Integrated graphics output showed BIOS after reset, but Ubuntu installer used discrete AI compute GPU video; monitor was moved temporarily.
- After system installation and shutdown, monitor attached to integrated motherboard video and dummy-display adapter attached to discrete GPU.
- Host booted but display connected to motherboard remained blank. Console routing to discrete GPU is a hypothesis, not yet proven.
- Requirement for all future installations/recovery: no user monitor cable switching. This acceptance test currently fails.

## Network and SSH
- Original installer DHCP address was not noted.
- Router UI showed alternating previous/current device names and different IP addresses. Router cause is suspected but unconfirmed; no router-side changes planned now.
- One address worked and SSH connectivity was stable. User decided to leave hardware in its current state and continue remotely.

## Checkpoint and pending evidence
- Owner-reported PASS: Ubuntu boot and stable LAN SSH.
- Owner-reported FAIL: no visible local Linux console via integrated GPU.
- UNKNOWN: current DHCP lease and identity, exact root filesystem, SSH hardening, graphics driver/DRM ownership, GPU compute, boot order, firewall and virtualization.
- Next action: read-only diagnostics over existing SSH, redacted before publication. Do not disable/blacklist GPU drivers or change network/firewall until the initial inventory is reviewed.
- Do not publish IP addresses, MAC addresses, usernames, router identifiers, credentials or raw unreviewed command outputs.
