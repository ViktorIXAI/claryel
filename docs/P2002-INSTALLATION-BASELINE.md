# P2002 — Initial Ubuntu Server Installation Baseline

**Status: owner-described installer decisions, 2026-11-06. Installation has not been independently verified. Not an authorization to erase any further disks.**

## Install order and choices
1. Owner-controlled BIOS/UEFI factory reset; inspect and re-enable required SVM/IOMMU, Secure Boot and storage settings after reset. Record only sanitized observations.
2. Boot from the dedicated 64 GB installer USB, intended to remain attached for a later independent recovery environment.
3. Ubuntu Server 24.04 LTS; **English** installation language. Minimal standard server packages.
4. Use **wired Ethernet with DHCP** from the router, initially allowing Internet access for setup and vetted packages. The built-in Wi-Fi module is not used; disable the interface if appropriate after verifying identification. No router port forwarding to SSH.
5. **Custom/manual storage layout.** Owner reports an internal approximately **2 TB drive**. The proposed final root partition is **400 GB, ext4, mounted at `/`, unencrypted** for the initial testbed. This supersedes the earlier 350 GB plan. Leave remaining capacity unallocated for later independent LUKS2 secrets and private-data volumes.
6. Under UEFI, select the intended internal NVMe as **boot device**, with a suitable EFI System Partition (ESP; size chosen/validated by installer) plus the 400 GB root partition. The ESP is distinct from root; do not infer a 2 GB disk size from an installer boot/partition display.
7. The owner intends to delete existing partitions on the selected target disk and reformat it. **This is destructive.** Before confirmation inspect model/capacity/identity and all pending changes. Verify this is the intended internal NVMe, not the 64 GB USB or another drive, and confirm that there is no required data. Do not perform any wipe without owner verification of target and data-loss acceptance. A partition reformat is not a verified full-device secure erase.
8. Create hostname **`p2002`** (Linux canonical lower case), a non-root administrative user, and a unique strong password. Do not publish the username/password, real IP, MAC, serial or other sensitive installation details. Choose **Install OpenSSH Server** only for LAN administration; subsequently restrict allowed source hosts, use SSH keys and disable password authentication after key-based access is validated.
9. Avoid optional packages/snaps/services and unnecessary SSH imported keys from unverified sources. Finish installation, reboot to internal NVMe (USB remains present) and verify network, disk layout, SSH, firmware settings and host identity.

## Security exceptions during this test phase
- The **400 GB root is unencrypted**, intentionally only for initial deployment. No real client documents, live credentials, secret keys, sensitive logs or production data may reside on it. Do not enable unencrypted swap containing sensitive data. Consider encrypted ephemeral swap or no swap until a protected solution is tested.
- A Wi-Fi module existing in the host is not considered a trusted extra management channel.
- Internet access during bootstrapping is temporary and controlled. Closed services are not authorized general Internet access merely because the host is online.
- Future root encryption is **only by verified backup → explicit destructive wipe → clean LUKS2 installation → tested restoration**, never in-place conversion. This is a future separately approved exercise.
- The recovery USB is not a trusted backup and should not be configured to auto-boot or auto-run restoration commands.

## Immediate post-install checks (read-only)
```bash
hostnamectl
ip -br addr
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
findmnt /
sudo systemctl is-active ssh
```
Record sanitized findings locally and publish only a cleaned, reviewed summary.

## Current intended layout (not verified)
- ESP: installer/UEFI-required size, on internal system disk;
- `/`: **400 GB ext4, unencrypted**;
- remainder: **unallocated**, intended for later independent LUKS2 secrets (~16 GB) and private-data volume;
- model/Strata storage budget **100 GB** within the system filesystem, with 32 GB swap planning subject to protection requirements.

All capacity numbers remain targets until the actual installer layout and `lsblk` are inspected.
