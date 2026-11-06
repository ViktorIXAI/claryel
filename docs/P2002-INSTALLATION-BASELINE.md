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

## Owner installation report: initial install completed, first reboot and display routing under test (2026-11-06)

**Owner observation, not independently verified:** Ubuntu Server installer displayed "Installation complete / Reboot now". The installer then requested removal of installation media; owner deliberately kept the 64 GB recovery/installer USB attached and pressed Enter. First reboot is being checked; later power-off and display cable relocation are planned. Do not mark reboot, SSH access, storage layout, or graphics routing PASS until actual evidence is recorded.

### Dual-GPU console behavior
- The P2002 motherboard has integrated graphics, and a discrete GPU is installed in the first PCIe slot for AI inference, not graphics workloads.
- After BIOS factory reset, firmware/BIOS output appeared on the display attached to motherboard video. During Ubuntu Server installation, console output switched to the discrete GPU, requiring temporary monitor reconnection.
- Owner plans to verify first reboot, then shut down, reconnect the physical monitor to motherboard integrated video output, and fit an existing HDMI/DP dummy-display emulator to the discrete GPU.
- **Unverified assumption:** the dummy plug does not guarantee that Linux console/fbcon will move to integrated graphics, or that discrete GPU VRAM use will be zero; test driver, kernel console and compute use separately.
- A future default procedure should keep internal NVMe first in UEFI boot order and invoke the one-time UEFI boot selection menu for USB recovery/install rather than permanently changing boot priority. The firmware-specific key and boot entries must be verified, not assumed.
- Check that boot does not automatically select the attached recovery USB, and document a recovery path if there is no local console; SSH access must also be tested.

### Suggested post-boot diagnostics (read-only, sanitize before public sharing)

```bash
hostnamectl
ip -br addr
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
lspci -nn | grep -Ei 'vga|3d|display'
cat /proc/cmdline
ls -l /dev/dri 2>/dev/null
systemctl is-active ssh
```

Avoid prematurely setting `nomodeset`, blacklisting the discrete GPU module, or forcibly binding/removing drivers: these changes may damage AMD compute availability for Strata or hide the console. Establish whether the selected integrated port receives Linux text output; headless SSH operation is an independent fallback. Physical replugging should be done after power-off, and no physical display dummy is a security control.

## Requirement for the NEXT install and recovery on equivalent hardware

The owner explicitly requires a **zero-monitor-cable-switch** workflow. A single monitor must remain attached to the motherboard integrated graphics for BIOS/UEFI, USB boot selection, Ubuntu installer, first boot, future offline recovery and encrypted reinstall. The AI compute GPU must never require the physical monitor. Test and record the required firmware settings and installer/recovery kernel console configuration; do not presume the current dummy-display emulator redirects the Linux console or yields zero VRAM usage. The previous P2002 install exposed the problem; a validated fix has not yet been demonstrated. Preserve an accessible local recovery console when networking is unavailable.
