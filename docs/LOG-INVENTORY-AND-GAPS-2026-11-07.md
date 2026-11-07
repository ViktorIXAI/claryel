# CLARYEL log inventory and evidence gaps — 2026-11-07

**Scope:** public GitHub repository ViktorIXAI/claryel, default branch main, plus visible issue #9 comments and recent commit history. **As-of repository snapshot:** refreshed 2026-11-07 after publication of the Strata inference checkpoint and the standing public-architecture disclosure direction. This is a repository snapshot, not live P2002 telemetry.

## 1. Repository log inventory

The current logs root contains only one machine directory: logs/P2002/. It contains **11 standalone Markdown records**. The 50 KB initial-install.md is the long chronological narrative; the other ten files are focused checkpoints.

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
| 10 | logs/P2002/2026-11-06-strata-iq3s-download-bringup.md | Strata 0.2.11/build 0.1.57, failed first launch, corrected Docker device passthrough retry | Historical checkpoint ends at GPU device nodes and engine check; superseded by later owner-supplied inference evidence in row 11, but not by a complete ROCm smoke-test transcript |
| 11 | logs/P2002/2026-11-07-strata-private-api-inference-success.md | Loaded Qwen3.8-Flash-Next IQ3_S, successful private API inference, native API-key source inspection, profile key status, direct host-to-container HTTP 200, bridge route, iptables and displayed Guard-table review | Owner-supplied transcript; API remains unauthenticated; host-to-container access confirmed; displayed Guard rules do not cover the inference bridge; LAN/WAN reachability and all-route authentication remain NOT_TESTED; reasoning_content review remains open |

## 2. Other evidence surfaces

- docs/AUDIT-STATUS-AND-EVIDENCE.md: refreshed status summary now includes the successful Strata/Qwen inference checkpoint; the separate raw ROCm/HIP smoke-test evidence remains missing.
- docs/EVIDENCE-INDEX-UPDATE-2026-11-06-02.md: links the backup and IQ3_S bring-up events; its last status is IN PROGRESS / NOT VERIFIED.
- Issue #9 has five visible comments after the 2026-11-07 inference update. The latest comment records successful model/API inference while keeping the separate raw ROCm/HIP smoke-test transcript open.
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
- Subsequent owner-supplied output shows Strata ready, the Qwen3.8-Flash-Next IQ3_S model loaded, health/model endpoints returning HTTP 200, and a disposable private-network client receiving the exact expected answer at 48.6 generated tokens/second (prompt processing 151.3 tokens/second). See log 11. This is functional inference evidence, not a security acceptance.

### Not evidenced in this repository snapshot
- Complete firmware package correction and final RX 9070 amdgpu binding chronology.
- Exact host-side kernel, firmware, driver and ROCm/HIP version matrix at the time of successful GPU test.
- rocminfo/amd-smi output identifying gfx1201 after the final fix.
- Actual HIP compute smoke-test command, exit code and output.
- Model shard/projector/drafter download completion, exact hashes and format validation.
- Full post-fix ROCm/HIP device-identification and standalone GPU smoke-test transcript (separate from the successful model inference evidence) is still not present in the public log snapshot.
- Post-reboot Cloud Hermes VM state, DHCP lease, SSH and autostart acceptance.
- A verified restore-from-backup boot.
- Machine-readable cross-file event inventory, global chronological checkpoint index, and reconciliation of every commit with a log checkpoint.

**Important:** owner reports that GPU testing works, and the new inference transcript confirms the model runs through the local Strata API. The remaining evidence gap is the exact post-fix ROCm/HIP device-identification and standalone GPU smoke-test transcript. Do not label the GPU failed; recover the raw terminal/agent evidence and append it.

## 4. Chronology and consistency issues

1. The initial install file accumulates many unrelated milestones; a separate checkpoint is preferable for each major subsystem.
2. The network acceptance file contains a proposed download command explicitly marked NOT EXECUTED, followed by a later separate actual bring-up log. The index must preserve this distinction.
3. The earlier issue #9 comments describe firmware investigation and corrected container device passthrough. The 2026-11-07 comment adds successful Strata/Qwen inference; standalone GPU compute acceptance remains a separate evidence gap.
4. The current audit-status page is a snapshot and must carry a last-evidence timestamp; it should not be treated as live status.
5. The 2026-11-06 temporary publication window is historical and expired. The owner issued a standing 2026-11-07 direction permitting public disclosure of P2002 non-secret architecture and lab identifiers; credentials, customer data and unrelated personal/network identifiers remain excluded. Per-client deployments must use generated unique identities and must not reuse P2002 runtime values.

## 5. Required next audit steps

- Recover latest GPU/Strata logs from P2002 host log directory, agent source logs, other branches and any owner-controlled local Git mirror.
- Create one event ID per material operation and map it to commit, file, issue/comment and machine identity.
- Add a machine-readable index (JSON/YAML) and a human-readable timeline; validate no orphan event or unsupported PASS.
- Add the successful inference checkpoint (now published), recover the separate GPU/ROCm smoke-test transcript and model artifact hashes, and add missing checkpoints for Cloud Hermes post-reboot state and backup restore when evidence becomes available.
- Verify Strata API authentication, host-port exposure, all peers on claryel-private-ai, direct-IP versus Host-header behavior, and reasoning_content response handling before expanding the set of agent clients.
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
