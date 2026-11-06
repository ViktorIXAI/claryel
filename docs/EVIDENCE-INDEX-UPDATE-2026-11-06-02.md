# Evidence index update — 2026-11-06 (append-only)

This file supplements, and does not rewrite, `AUDIT-STATUS-AND-EVIDENCE.md`. It records the latest owner-supplied P2002 checkpoint after the previous index snapshot.

## Latest P2002 backup evidence

1. [Pre-reboot backup preflight safe-stop](../logs/P2002/2026-11-06-pre-reboot-backup-preflight-aborted.md) — the backup script stopped safely because a target XML filename already existed. Existence was not treated as proof of validity.
2. [Pre-reboot backup inventory and validation](../logs/P2002/2026-11-06-pre-reboot-backup-inventory.md) — owner-reported diagnostic found:
   - both XML backups parse and byte-match their current inactive libvirt definitions;
   - the Guard boot archive passes gzip integrity validation and contains the seven listed configuration files;
   - all three qcow2 backup images pass `qemu-img check`;
   - guest state is `shut off`, the Guard verifier passes, and SSH is active;
   - `pre-reboot-state.txt` is the only missing item in the inspected inventory.
3. [Earlier guest-image byte-identity verification](../logs/P2002/2026-11-06-pre-reboot-backup-verification.md) remains a separate checkpoint. Its byte comparison and qcow2 check do not establish restore bootability.

## Current gate

**Do not overwrite or recreate any existing backup.** Create only the missing `pre-reboot-state.txt` with restrictive permissions and a no-overwrite check. Then, subject to the owner's separate execution approval, perform the controlled host reboot and inspect the actual service activation chronology. DHCP, guest boot, packet-level isolation, reboot persistence and restore boot remain NOT_TESTED until evidence is collected.

## Scope and provenance

- Evidence source: OWNER_REPORTED terminal output supplied in the active conversation.
- Review: SELF_REVIEW_ONLY; no independent shell access to P2002.
- Writer-instance: CHATGPT-REVIEW-01.
- Writer-role: REPOSITORY_AUDITOR.
- Hermes-instance-ID: NOT_APPLICABLE (this entry is authored by the ChatGPT review process, not a Hermes instance).
- Hermes-runtime-machine-ID: NOT_APPLICABLE.
- Target-machine-ID / deployment alias: P2002.
- Executed-by: OWNER_REPORTED_OPERATOR; actual shell operator identity is not independently verified.
- Published-by: GitHub connector using the shared ViktorIXAI account; shared GitHub identity is not individual attribution proof.
- Related open requirements: [#1 publication gate](https://github.com/ViktorIXAI/claryel/issues/1), [#2 private evidence ledger](https://github.com/ViktorIXAI/claryel/issues/2), [#3 checkpoint schema](https://github.com/ViktorIXAI/claryel/issues/3), [#5 evidence index](https://github.com/ViktorIXAI/claryel/issues/5), [#6 writer attribution](https://github.com/ViktorIXAI/claryel/issues/6).

## Explicitly not claimed

No reboot has occurred in this checkpoint; no Hermes runtime has been installed or started; no restore boot has been tested; and the Guard-table-loss-after-Docker-startup experiment remains deferred.


## Latest P2002 Strata model-download checkpoint — 2026-11-06

4. [Strata IQ3_S download bring-up log](../logs/P2002/2026-11-06-strata-iq3s-download-bringup.md) — append-only record of the CLI investigation, first failed launch, and GPU-device-passthrough retry.
   - The existing [network isolation acceptance](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md) already records the full firewall/Squid/Guard candidate, verifier corrections, production deployment, UFW reload, positive/negative container tests, real reboot, backup discrepancy and the earlier **unexecuted** proposed download command.
   - The first actual `strata launch` attempt failed with exit code 5 before any model files were written. Its diagnostic reported missing model shards, prepared IQ3_S pack and tokenizer. The first Docker run omitted `/dev/kfd` and `/dev/dri`.
   - The retry passed `/dev/kfd` and `/dev/dri` and the available render/video groups. The replacement container started and showed those device nodes inside the container; the last supplied log was still checking the engine for this PC.
   - **Current status: IN PROGRESS / NOT VERIFIED.** No GGUF shard, mmproj, DFlash2 file, prepared pack or tokenizer has yet been confirmed downloaded or checksum-verified. No inference result is claimed.
   - The earlier `strata install -- --help` attempt reported that the image directory has no `setup.py`; the subsequent help commands in that shell block did not execute because `set -e` stopped at that error.
   - Continue by inspecting the existing detached container and volume without removing or restarting the container while it may still be working. Keep the explicit proxy allowlist; do not widen it to a wildcard.

