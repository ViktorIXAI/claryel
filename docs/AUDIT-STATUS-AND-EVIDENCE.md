# CLARYEL implementation evidence status

**As of 2026-11-06 (documentation review).** This is a source-index and gap assessment, not live host telemetry or an independent security certification. It must be refreshed from evidence; do not infer execution state from Git commit titles alone.

## Current scope

- Architecture: design proposal; isolation, security gateway, local inference and offline recovery have not passed end-to-end acceptance.
- P2002: laboratory host installation and incremental security/network validation are **in progress**, per owner-supplied public narrative.
- Cloud Hermes: guest disk networking configuration was edited offline after verified image backups; a persistent VirtIO management interface was attached to the isolated libvirt network. The latest P2002 owner-supplied checkpoint records UFW/Guard source inspection. The VM was last reported shut off; DHCP after boot, UFW/Guard boot ordering, packet-level isolation and reboot persistence remain **NOT_TESTED**.
- Hardware benchmark comparisons and production/customer use: not established by this evidence index.

## Evidence and issues

| Area | Current public evidence | Limitation | Follow-up |
| --- | --- | --- | --- |
| Installation sequence | [P2002 journal](../logs/P2002/2026-11-06-initial-install.md) | Owner-provided observations, not live independent shell audit | [#3](https://github.com/ViktorIXAI/claryel/issues/3) |
| Pre-reboot guest-image checkpoint | [Backup verification](../logs/P2002/2026-11-06-pre-reboot-backup-verification.md) | Existing backup byte-identical to source; qcow2 check clean; guest shut off. Restore boot and host reboot ordering NOT_TESTED. | [#3](https://github.com/ViktorIXAI/claryel/issues/3), [#5](https://github.com/ViktorIXAI/claryel/issues/5) |
| Firewall validation | Same journal: exact JSON checker and simulated missing DROP rule | No universal packet-level or boot-persistence proof | [#4](https://github.com/ViktorIXAI/claryel/issues/4) |
| UFW / Guard integration | [UFW/Guard source](../logs/P2002/2026-11-06-ufw-guard-audit.md), [bridge inspection](../logs/P2002/2026-11-06-ufw-guard-boot-timing.md), [observed activation chronology](../logs/P2002/2026-11-06-systemd-activation-order.md), [effective dependency inspection](../logs/P2002/2026-11-06-effective-guard-dependencies.md) | Current effective graph reports Requires+After Guard for Docker and libvirt; Guard ordered Before both. UFW interactions, failure after startup, packet-level isolation and next reboot NOT_TESTED. | [#1](https://github.com/ViktorIXAI/claryel/issues/1), [#4](https://github.com/ViktorIXAI/claryel/issues/4) |
| Publication safety | [Two-ledger policy](../policies/PUBLICATION-SECURITY.md) | Mandatory prepublication enforcement not verified | [#1](https://github.com/ViktorIXAI/claryel/issues/1) |
| Private evidence | Two-ledger policy | Protected full-fidelity ledger not deployed | [#2](https://github.com/ViktorIXAI/claryel/issues/2) |
| Project status | [README](../README.md) | Not production-ready; documentation subject to concurrent changes | [#5](https://github.com/ViktorIXAI/claryel/issues/5) |

## Reading rules

1. Git history proves a publication history, not that every described shell action occurred.
2. Explicitly distinguish OWNER_REPORTED, INDEPENDENTLY_TESTED, SIMULATED, PROPOSED and UNKNOWN.
3. A passing firewall policy inspection does not demonstrate traffic isolation, compromise resistance, or persistence after reboot.
4. Never rewrite the ongoing P2002 installation journal as part of editorial review; use new artifacts and linked corrections.
5. Check the latest chronological checkpoint before taking installation action. This index cannot authorize operations.

For future events use [the checkpoint template](../templates/AUDIT-CHECKPOINT.md). No production claims are made here.
