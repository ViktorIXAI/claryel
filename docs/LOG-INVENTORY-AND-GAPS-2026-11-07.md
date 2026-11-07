# CLARYEL log inventory and evidence gaps — 2026-11-07

**Scope:** public GitHub repository ViktorIXAI/claryel, default branch main, plus visible issue #9 comments and recent commit history. **As-of repository snapshot:** latest commit observed was 7513979be541a49443ae947b0a0ddc4197f3492d at 2026-11-06T21:34:22Z. This is a repository snapshot, not live P2002 telemetry.

## 1. Repository log inventory

The current logs root contains only one machine directory: logs/P2002/. It contains **10 standalone Markdown records**, not 11. The 50 KB initial-install.md is the long chronological narrative; the other nine files are focused checkpoints.

| # | File | Main scope | Evidence limitation / gap |
| --- | --- | --- | --- |
| 1 | logs/P2002/2026-11-06-initial-install.md | Initial Ubuntu install, SSH, Docker, zone probes, KVM/VM, Guard build and offline guest network inspection | Long narrative; later GPU/Strata sequence is not included in full |
| 2 | logs/P2002/2026-11-06-ufw-guard-audit.md | UFW unit, Guard loader/verifier source, firewall restore/flush search | Several operations (UFW stop/restart, packet isolation, IPv6) NOT_TESTED |
| 3 | logs/P2002/2026-11-06-ufw-guard-boot-timing.md | UFW/Guard boot timing, nft bridge inspection | Does not prove later-installed dependency behavior on reboot |
| 4 | logs/P2002/2026-11-06-systemd-activation-order.md | Observed current-boot activation chronology | Dependencies were installed after initial service starts; chronology is not proof of effective ordering on that boot |
| 5 | logs/P2002/2026-11-06-effective-guard-dependencies.md | Effective systemd Requires/After properties | Does not prove runtime behavior after Guard table loss; BindsTo empty |
| 6 | logs/P2002/2026-11-06-pre-reboot-backup-verification.md | Existing qcow2 byte identity, SHA-256 match and qemu-img check | Restore boot and host reboot ordering NOT_TESTED |
| 7 | logs/P2002/2026-11-06-pre-reboot-backup-preflight-aborted.md | Safe refusal to overwrite existing backup | Correctly records ABORTED; not a backup-success checkpoint |
| 8 | logs/P2002/2026-11-06-pre-reboot-backup-inventory.md | XML, archive, qcow2 and Guard inventory | pre-reboot-state.txt missing; restore remains NOT_TESTED |
| 9 | logs/P2002/2026-11-06-strata-network-isolation-acceptance.md | Restricted egress, Squid, positive/negative TCP/HTTPS tests, one reboot | Bounded tests only; IPv6, DNS/UDP/QUIC, L2, Guard-loss and second reboot NOT_TESTED |
| 10 | logs/P2002/2026-11-06-strata-iq3s-download-bringup.md | Strata 0.2.11/build 0.1.57, failed first launch, corrected Docker device passthrough retry | Last recorded state is GPU device nodes visible and Strata checking the engine; no final ROCm compute/model/inference result |

## 2. Other evidence surfaces

- docs/AUDIT-STATUS-AND-EVIDENCE.md: status summary, but its GPU/Strata status mirrors the last partial bring-up checkpoint and is stale if later successful tests occurred outside this public repository snapshot.
- docs/EVIDENCE-INDEX-UPDATE-2026-11-06-02.md: links the backup and IQ3_S bring-up events; its last status is IN PROGRESS / NOT VERIFIED.
- Issue #9 has four visible comments. The latest comment repeats the partial retry status and says no ROCm smoke test, model download, model load or inference was yet confirmed.
- Commit history contains numerous checkpoint commits, but a commit is not a substitute for the corresponding execution log.
- Current logs root contains only P2002. No P1005 log directory was found.
- The adjacent claryel-autonomous repository is a separate AI-agent benchmark/research project; do not treat its extensive experiment logs as P2002 installation logs.

## 3. Confirmed evidence versus gaps

### Confirmed in repository records (owner-supplied, not independently executed by this reviewer)
- Host/virtualization baseline, Docker zone-separation probes and cleanup.
- Guard policy verifier and a simulated negative policy-tamper test against an in-memory JSON copy.
- Backup checks and one real reboot chronology.
- Bounded post-reboot restricted proxy positive/negative tests.
- Strata 0.2.11 / engine build 0.1.57 identification.
- First Strata launch failed before model download; corrected retry passed /dev/kfd and /dev/dri into container and observed device nodes there.

### Not evidenced in this repository snapshot
- Complete firmware package correction and final RX 9070 amdgpu binding chronology.
- Exact host-side kernel, firmware, driver and ROCm/HIP version matrix at the time of successful GPU test.
- rocminfo/amd-smi output identifying gfx1201 after the final fix.
- Actual HIP compute smoke-test command, exit code and output.
- Model shard/projector/drafter download completion, exact hashes and format validation.
- Strata successful model preparation, API readiness, model load and verified inference response.
- Subsequent GPU/container tests reportedly performed by the owner but not represented in the current public log or issue #9 comments.
- Post-reboot Cloud Hermes VM state, DHCP lease, SSH and autostart acceptance.
- A verified restore-from-backup boot.
- Machine-readable cross-file event inventory, global chronological checkpoint index, and reconciliation of every commit with a log checkpoint.

**Important:** owner reports that GPU testing works. This inventory does not dispute that report; it records that the matching evidence has not been found in the public repository snapshot reviewed. Do not label the GPU failed. Recover the exact later terminal transcript/agent log and append a correction/update.

## 4. Chronology and consistency issues

1. The initial install file accumulates many unrelated milestones; a separate checkpoint is preferable for each major subsystem.
2. The network acceptance file contains a proposed download command explicitly marked NOT EXECUTED, followed by a later separate actual bring-up log. The index must preserve this distinction.
3. The initial issue #9 comments describe firmware investigation and later corrected container device passthrough, but not final successful GPU compute/inference acceptance.
4. The current audit-status page is a snapshot and must carry a last-evidence timestamp; it should not be treated as live status.
5. The 24-hour fast-log publication exception has an absolute expiry; all agents must check the current effective policy before publication. The exception does not waive secret, private-data or material infrastructure-exposure controls.

## 5. Required next audit steps

- Recover latest GPU/Strata logs from P2002 host log directory, agent source logs, other branches and any owner-controlled local Git mirror.
- Create one event ID per material operation and map it to commit, file, issue/comment and machine identity.
- Add a machine-readable index (JSON/YAML) and a human-readable timeline; validate no orphan event or unsupported PASS.
- Add missing checkpoints for GPU/ROCm, model download/integrity, model preparation/inference, Cloud Hermes post-reboot state, and backup restore when evidence becomes available.
- Preserve old records; append corrections rather than rewriting history.

## 6. Related tasks

- [#9 RX 9070 / Strata bring-up](https://github.com/ViktorIXAI/claryel/issues/9)
- [#10 Cloud Hermes autostart](https://github.com/ViktorIXAI/claryel/issues/10)
- [#15 complete log inventory and evidence recovery](https://github.com/ViktorIXAI/claryel/issues/15)
- [#11 artifact supply-chain admission](https://github.com/ViktorIXAI/claryel/issues/11)
- [#12 fail-closed lifecycle](https://github.com/ViktorIXAI/claryel/issues/12)

## Provenance

Writer-instance: CHATGPT-REVIEW-01
Writer-role: REPOSITORY_AUDITOR
Deployment-alias: P2002-LAB
Prepared-by: ChatGPT GPT-6 (declared)
Executed-by: NOT_APPLICABLE (GitHub read/write documentation only)
Evidence-source: TOOL_OBSERVED public repository tree, files, commit history and issue #9 comments; OWNER_REPORTED GPU-success assertion
Reviewed-by: SELF_REVIEW_ONLY
Approved-by: owner request for repository inventory
Published-by: GitHub connector via shared ViktorIXAI account
Attribution-confidence: DECLARED_UNVERIFIED
