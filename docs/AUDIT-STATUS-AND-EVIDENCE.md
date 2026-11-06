# CLARYEL implementation evidence status

**As of 2026-11-06 (documentation review).** This is a source-index and gap assessment, not live host telemetry or an independent security certification. It must be refreshed from evidence; do not infer execution state from Git commit titles alone.

## Current scope

- **Architecture:** design proposal; the full cross-zone system, Security GTW, local inference, offline recovery and end-to-end acceptance are not complete.
- **P2002:** laboratory host installation and incremental security/network validation are **in progress**. The owner supplied a post-reboot functional acceptance transcript for the restricted Strata proxy path and a subsequent model-download bring-up transcript; see the [P2002 network isolation and proxy acceptance log](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md) and [IQ3_S download bring-up log](../logs/P2002/2026-11-06-strata-iq3s-download-bringup.md). The network result is a bounded functional test, not a security certification.
- **Cloud Hermes:** guest disk networking configuration was edited offline after verified image backups; a persistent VirtIO management interface was attached to the isolated libvirt network. The latest network-acceptance transcript did not inspect the VM state after the reboot. Do not infer VM autostart or guest DHCP success from the Guard/Docker/Squid boot sequence; see [issue #10](https://github.com/ViktorIXAI/claryel/issues/10).
- **P2002 GPU / Strata:** owner-supplied post-reboot output confirms host `/dev/kfd` and `/dev/dri` device nodes and shows them passed into a Strata container; Strata reports HIP engine 0.1.57 and had reached helper-package / computer checks at the last supplied log. This is not yet a successful `rocminfo`/GPU-compute smoke test. The first launch without device passthrough failed with exit code 5 and missing model shards/pack/tokenizer; the corrected detached container was still running its checks. No GGUF shard, mmproj, DFlash2, prepared pack, tokenizer, model load or inference has yet been confirmed. Issue #9 remains open; see the [IQ3_S bring-up log](../logs/P2002/2026-11-06-strata-iq3s-download-bringup.md).
- **Hardware benchmark comparisons and production/customer use:** not established by this evidence index.

## Evidence and issues

| Area | Current public evidence | Limitation | Follow-up |
| --- | --- | --- | --- |
| Installation sequence | [P2002 journal](../logs/P2002/2026-11-06-initial-install.md) | Owner-provided observations, not live independent shell audit | [#3](https://github.com/ViktorIXAI/claryel/issues/3) |
| Pre-reboot guest-image checkpoint | [Backup verification](../logs/P2002/2026-11-06-pre-reboot-backup-verification.md) | Existing backup byte-identical to source; qcow2 check clean; guest shut off. Restore boot and independent backup recovery remain unverified. | [#3](https://github.com/ViktorIXAI/claryel/issues/3), [#5](https://github.com/ViktorIXAI/claryel/issues/5) |
| Strata network isolation and proxy persistence | [Post-reboot acceptance log](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md) | Owner-run positive/negative TCP and HTTPS CONNECT tests passed after one reboot. IPv6, DNS/UDP/QUIC, L2, SSRF edge cases, Guard loss/tampering, second reboot and service-failure recovery remain NOT_TESTED. | [#4](https://github.com/ViktorIXAI/claryel/issues/4), [#7](https://github.com/ViktorIXAI/claryel/issues/7) |
| UFW / Guard integration | [UFW/Guard source](../logs/P2002/2026-11-06-ufw-guard-audit.md), [bridge inspection](../logs/P2002/2026-11-06-ufw-guard-boot-timing.md), [observed activation chronology](../logs/P2002/2026-11-06-systemd-activation-order.md), [effective dependency inspection](../logs/P2002/2026-11-06-effective-guard-dependencies.md), and [post-reboot proxy acceptance](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md) | One UFW reload preserved the live proxy exception. This does not prove every UFW operation or every firewall manager preserves the Guard. | [#1](https://github.com/ViktorIXAI/claryel/issues/1), [#4](https://github.com/ViktorIXAI/claryel/issues/4), [#7](https://github.com/ViktorIXAI/claryel/issues/7) |
| Backup integrity | [Pre-reboot backup records](../logs/P2002/2026-11-06-pre-reboot-backup-inventory.md) and [network acceptance backup audit](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md) | Guard deployment backup checksums passed. Proxy persistence backup contains iptables/nftables/Squid snapshots but lacks a SHA256SUMS manifest; its manifest is still outstanding. | [#2](https://github.com/ViktorIXAI/claryel/issues/2), [#5](https://github.com/ViktorIXAI/claryel/issues/5) |
| RX 9070 / Strata / Qwen model | [Network acceptance](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md); [IQ3_S download bring-up](../logs/P2002/2026-11-06-strata-iq3s-download-bringup.md); [issue #9](https://github.com/ViktorIXAI/claryel/issues/9) | Host and container GPU device-node visibility is owner-reported; ROCm enumeration/compute is not yet proven. First launch failed before download; corrected container was still checking the engine in the last supplied output. No model files or inference have been verified. | [#9](https://github.com/ViktorIXAI/claryel/issues/9) |
| Publication safety | [Two-ledger policy](../policies/PUBLICATION-SECURITY.md) | Mandatory prepublication enforcement and exact-byte scanning are not verified. This manual review is not equivalent to the required fail-closed gate. | [#1](https://github.com/ViktorIXAI/claryel/issues/1) |
| Private evidence | Two-ledger policy | Protected full-fidelity ledger and recovery checks are not deployed. | [#2](https://github.com/ViktorIXAI/claryel/issues/2) |
| Project status | [README](../README.md) | Not production-ready; documentation subject to concurrent changes. | [#5](https://github.com/ViktorIXAI/claryel/issues/5) |

## Reading rules

1. Git history proves a publication history, not that every described shell action occurred.
2. Explicitly distinguish OWNER_REPORTED, TOOL_OBSERVED, INDEPENDENTLY_VERIFIED, SIMULATED, PROPOSED and UNKNOWN.
3. A passing policy verifier is not a traffic test. A passing bounded traffic test is not proof against every protocol or bypass.
4. Preserve mistakes, failed hypotheses, aborted attempts and corrections in new dated artifacts; do not silently rewrite the ongoing P2002 installation journal.
5. Owner-supplied terminal output is not independent execution by the documentation author.
6. Check the latest chronological checkpoint and unresolved issues before taking installation action. This index cannot authorize operations.

For future events use [the checkpoint template](../templates/AUDIT-CHECKPOINT.md). No production or security-certification claims are made here.
