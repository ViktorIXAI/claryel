# 64 GB Persistent Recovery USB — Architecture Standard

**Status: owner-directed architecture requirement, 2026-11-06. Planned, not yet implemented, secured or tested.** Applies as a design target to each future CLARYEL box; specifics require device-level validation.

## Bootstrapping order on P2002

1. Before resetting firmware, inventory the existing BIOS/UEFI state and record non-secret essentials, including boot mode, storage controller / RAID configuration, SATA/NVMe visibility, virtualization (SVM/IOMMU), Secure Boot, TPM and GPU settings. Check that a factory reset will not strand encrypted/RAID data or remove essential access. A factory default reset is then performed **by the owner** and rechecked for required settings; defaults are not presumed secure.
2. Use a **64 GB USB flash drive** prepared from verified official Ubuntu Server 24.04 installation media. Validate ISO authenticity/checksum where possible. Boot from USB and install the initial test system on the intended **350 GB unencrypted NVMe system area only**. Confirm disk and partition selection carefully; never install the OS onto the recovery USB by accident.
3. Leave the 64 GB USB physically connected after installing Ubuntu. Under normal operation the host should boot the internal NVMe, **not** silently boot an untrusted USB device. Record UEFI boot order and provide an explicit physical/admin-authorized recovery boot procedure.
4. **Later milestone:** build and validate a bootable, self-contained Linux recovery/test environment on that USB, operational independently of the main NVMe. Retain or recreate verified installation media as part of this design. Exact bootloader, partition sizes/filesystems and persistence strategy are to be tested, not guessed during initial deployment.
5. Reserve a separated, limited area of the USB for **non-sensitive restoration metadata**, e.g. versioned sanitized installation manifests, checksums, signed recovery instructions, disk-layout templates, restore scripts and repository revision IDs. Do not put live secrets, raw LUKS keys, private SSH keys, credentials, client documents or unencrypted backups there.
6. Later test the integrated sequence: independent verified backup -> non-destructive test restore -> explicit owner approval of target disk and wipe -> freshly install LUKS2-encrypted system -> recover settings, services and synthetic workloads -> verify offline restoration, bootability, GTW isolation, protected swap and encrypted private/secrets volumes.

## Security and reliability boundaries

- **Connected USB is an attack surface.** A malicious replacement or modified recovery environment could expose credentials and disks. A physically connected USB is not considered inherently trusted or safe simply because it is 'recovery media'. Document firmware/UEFI Secure Boot strategy, boot-chain integrity and media provenance; verify integrity before each privileged recovery use.
- Prefer normally **unmounted / read-only** recovery partitions and narrowly authorized maintenance operations. Automatic mounting, auto-executing restore scripts or unattended boot from USB are forbidden by default.
- USB flash wear, power loss and corruption are realistic failure modes. Keep **at least one independent recovery image or copy on separate media**; a USB continuously attached to the box is not a sufficient backup and does not protect against theft, fire, firmware compromise or simultaneous disk/media failure.
- A 64 GB flash drive **cannot** hold complete backups of a 350 GB system plus a ~78 GB model. It is a boot/recovery/control medium, **not the principal backup destination**. Full backups must be held on independently protected storage, designed later under the deferred backup milestone.
- Recovery tooling must remain capable of offline use. Do not depend on cloud auth, online package downloads or GitHub availability during a recovery test.
- Firmware factory defaults can disable SVM/IOMMU, Secure Boot and other security features or change boot and storage settings. Re-enable and verify each required security feature. Do not publish actual device serials, real host identifiers or private network details.
- An unattended system reset or wipe is never permitted. Require local owner-controlled approval, verified backup and a tested restore before destructive operations.
- For today's work, do not re-partition/reformat the existing installer USB after installation merely to create the recovery environment. Recovery USB conversion is a separately tested stage; preserve a known-good installer first.

## Proposed acceptance tests

- After factory reset the hardware still boots correctly with required virtualization and protection settings explicitly audited.
- Ubuntu installs onto NVMe while USB remains intact; normal reboot chooses NVMe even with USB connected.
- Subsequent offline USB boot works independently of NVMe, including with the internal drive disabled/disconnected in a controlled test.
- Recovery environment cannot read protected secrets/private data without explicit owner-supplied unlock credentials; changes are audited.
- Metadata partition can be inspected, compared and restored without internet; its integrity is verifiable.
- Independently stored backup restores a full test installation, including model artifacts and service policies, onto a freshly encrypted system. No customer data is used.

**Current state:** requirement documented only. No claim that the USB is already a functioning recovery system or validated backup.

## Mandatory display routing acceptance for next installation and recovery

**Owner requirement:** On systems with motherboard integrated graphics and a dedicated AI compute GPU, the user must never need to physically move a display cable between the GPUs during the installer, normal boot, or boot from recovery media. The monitor stays connected to the motherboard video port throughout; the compute GPU must remain available for AI. The current P2002 installation needed a workaround; this requirement is **NOT YET MET OR TESTED**.

Test firmware primary-display/iGPU-multimonitor settings and Linux installer/recovery console routing (including DRM/framebuffer, kernel and installer boot options), comparing screenshots/console visibility at UEFI, boot menu, installer, system login and offline recovery. Do not assume an HDMI dummy plug solves console assignment or frees all compute VRAM. Check that any driver/console change does not disable AI GPU compute. Prefer a prevalidated recovery USB image with deterministic display routing and a documented local fallback; a remote SSH recovery option alone is insufficient when networking fails. Preserve the normal NVMe-first and one-time USB boot policy. Record platform-specific BIOS/UEFI settings for a reproducible, low-touch setup.
