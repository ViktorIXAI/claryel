# CLARYEL log inventory and evidence gaps — 2026-11-07

**Scope:** public GitHub repository ViktorIXAI/claryel, default branch main, plus visible issue #9 comments and recent commit history. **As-of repository snapshot:** refreshed 2026-11-07 after publication of the Strata inference checkpoint and the standing public-architecture disclosure direction. This is a repository snapshot, not live P2002 telemetry.

## 1. Repository log inventory

The current logs root contains only one machine directory: logs/P2002/. It contains **16 standalone Markdown records**. The 50 KB initial-install.md is the long chronological narrative; the other fifteen files are focused checkpoints.

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
| 11 | logs/P2002/2026-11-07-strata-private-api-inference-success.md | Loaded Qwen3.8-Flash-Next IQ3_S and successful inference; exact image/container baseline; confirmed root UID/GID and mounts; full native API_KEY_FILE flow; private run-config writer uses chmod 0600 and atomic replace; server _authorized() helper accepts Bearer/x-api-key | Owner-supplied transcript; preset serializer/path/mode and /data backup scope remain unknown; filtered find did not inspect active profile basename; direct-serve persistence/logging and route call-site coverage unproven; dummy-key test, LAN/WAN reachability, all-route authentication and reasoning_content review remain open |

| 12 | logs/P2002/2026-11-07-cloud-hermes-agent-existing-correction.md | Owner correction that the existing guest Hermes installation and Grok response tests must be preserved; do not reinstall | Owner-reported prior completion; original installation transcript and exact service state were not present at that checkpoint |
| 13 | logs/P2002/2026-11-07-cloud-hermes-autostart.md | Libvirt network and Cloud Hermes VM autostart enabled; later addendum corrects the proposed reinstall step | No host reboot or guest application readiness test in that checkpoint |
| 14 | logs/P2002/2026-11-07-cloud-hermes-guest-runtime-discovery.md | Guest SSH, existing Hermes installation context, prior-conversation recovery and read-only runtime discovery | Earlier snapshot found no matching running process/unit and linger disabled; later Gateway installation supersedes that service-state snapshot; reboot persistence remains untested |
| 15 | logs/P2002/2026-11-07-host-boot-state-snapshot.md | Host boot/service state snapshot | Snapshot only; not a substitute for a later reboot acceptance test |
| 16 | logs/P2002/2026-11-07-cloud-hermes-network-pm-repair-gateway.md | Guest DNS/default-route diagnosis; Guard/libvirt/proxy separation; offline UV wheelhouse repair using hash-verified original packaging/setuptools; successful Hermes PM repair; Gateway installation, linger and SIGKILL auto-restart test | Guest remains without default route/upstream DNS; no messaging platforms enabled; auxiliary xAI OAuth and Nous credentials unavailable; actual logout/reboot persistence and end-to-end messaging NOT_TESTED |

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
- Subsequent owner-supplied output shows Strata ready, the Qwen3.8-Flash-Next IQ3_S model loaded, health/model endpoints returning HTTP 200, and a disposable private-network client receiving the exact expected answer at 48.6 generated tokens/second (prompt processing 151.3 tokens/second). Later read-only inspection found the profile has no API key, host-to-container HTTP 200 via the inference bridge, and no explicit inference-bridge rule in the displayed Guard table. A further container inspect snapshot recorded the immutable image digest, GPU devices, supplemental groups, volumes, restart policy and healthcheck. A subsequent corrected read-only diagnostic confirmed image and active process UID/GID 0:0, supplementary groups 0,44,993, the tini/entrypoint command, profile and mount-root permissions, and Docker-configured environment variable names with `STRATA_API_KEY` absent. The complete installed entrypoint excerpt shows API_KEY_FILE -> STRATA_LAUNCH_KEY -> private preset and derived run-config api_key, followed by a further key write to /data/state/api-key under umask 077. The run-config writer uses chmod 0600 (best effort) and atomic replacement; launch_dir() resolves under paths.state_base()/launch. Preset serializer/path/mode and /data backup scope remain unknown. The latest filename-filtered find returned two runtime logs at 0600 root:root but did not match the active profile basename iq3_s-128k.json, so it is not a recursive audit. The live container overrides the image entrypoint and directly runs strata serve --profile; this server accepts STRATA_API_KEY or profile api_key. An _authorized() helper checks Bearer/x-api-key and returns 401, but route call-site coverage and direct-serve persistence/logging are not yet established. A wrapper remains only a candidate pending these reviews and disposable dummy-key validation. See log 11. This is functional inference evidence plus bounded exposure/configuration findings, not a security acceptance.

### Not evidenced in this repository snapshot
- Complete firmware package correction and final RX 9070 amdgpu binding chronology.
- Exact host-side kernel, firmware, driver and ROCm/HIP version matrix at the time of successful GPU test.
- rocminfo/amd-smi output identifying gfx1201 after the final fix.
- Actual HIP compute smoke-test command, exit code and output.
- Model shard/projector/drafter download completion, exact hashes and format validation.
- Full post-fix ROCm/HIP device-identification and standalone GPU smoke-test transcript (separate from the successful model inference evidence) is still not present in the public log snapshot.
- Post-reboot Cloud Hermes VM state, DHCP lease and Gateway startup acceptance. Guest SSH and the systemd-managed Gateway were observed running before a reboot; post-reboot behavior remains NOT_TESTED.
- A verified restore-from-backup boot.
- Machine-readable cross-file event inventory, global chronological checkpoint index, and reconciliation of every commit with a log checkpoint.

**Important:** owner reports that GPU testing works, and the new inference transcript confirms the model runs through the local Strata API. The remaining evidence gap is the exact post-fix ROCm/HIP device-identification and standalone GPU smoke-test transcript. Do not label the GPU failed; recover the raw terminal/agent evidence and append it.

## 4. Chronology and consistency issues

1. The initial install file accumulates many unrelated milestones; a separate checkpoint is preferable for each major subsystem.
2. The network acceptance file contains a proposed download command explicitly marked NOT EXECUTED, followed by a later separate actual bring-up log. The index must preserve this distinction.
3. The earlier issue #9 comments describe firmware investigation and corrected container device passthrough. The 2026-11-07 comment adds successful Strata/Qwen inference; standalone GPU compute acceptance remains a separate evidence gap.
4. The current audit-status page is a snapshot and must carry a last-evidence timestamp; it should not be treated as live status.
5. The 2026-11-06 temporary publication window is historical and was superseded on 2026-11-07 by the standing owner instruction in the publication and installation-audit policies. Routine CLARYEL engineering logs are to be published promptly; do not wait for automated publisher, scanner or CI readiness. Non-secret P2002/CLARYEL lab network and operational details are authorized for publication. Credentials, customer/third-party confidential data and unrelated personal information remain excluded. Per-client deployments must use generated unique identities and must not reuse P2002 runtime values.

## 4.1 Cloud Hermes guest and Gateway checkpoint added — 2026-11-07

New record: `logs/P2002/2026-11-07-cloud-hermes-network-pm-repair-gateway.md`.

- Guest `172.30.240.12/24` on `claryel-cloud-admin` / `virbr240` has no default route; systemd-resolved is active but the link has no DNS scope/upstream server.
- Host Guard rules explicitly drop forwarding into and out of `virbr240`. The Squid proxy at `172.30.250.1:3128` is bound to the separate `br-strata-eg` network; it is not an allowance for the Cloud Hermes guest.
- No network/Guard change was made. The Hermes package manager was repaired offline from a local wheelhouse after original packaging 26.3 and setuptools 84.1.0 artifacts were verified against the guest lock-file hashes.
- Hermes Gateway is installed as an enabled user systemd service; `Linger=yes`.
- A controlled SIGKILL test passed: PID 2969 was restarted as PID 3132; `NRestarts=1`.
- Gateway is **not yet messaging-ready**: the log says `No messaging platforms enabled`. Auxiliary xAI OAuth and Nous Portal warnings are present. End-to-end messaging and reboot persistence remain NOT_TESTED.
- The earlier guest-runtime snapshot that found no matching service and linger disabled predates this later installation; preserve both checkpoints as chronological evidence.

## 5. Required next audit steps

- Recover latest GPU/Strata logs from P2002 host log directory, agent source logs, other branches and any owner-controlled local Git mirror.
- Preserve the Cloud Hermes guest isolation boundary; if package/network access is needed, design an explicitly approved broker/proxy path rather than adding an unreviewed default route or NAT.
- Inspect Hermes Gateway CLI help and non-secret messaging configuration, identify the intended messaging platform, and run a controlled inbound/outbound test. Verify the primary Grok session separately from auxiliary xAI OAuth/Nous warnings.
- After a planned maintenance window, test guest/host reboot recovery and confirm Gateway startup with linger; do not infer reboot acceptance from the SIGKILL restart test.
- Create one event ID per material operation and map it to commit, file, issue/comment and machine identity.
- Add a machine-readable index (JSON/YAML) and a human-readable timeline; validate no orphan event or unsupported PASS.
- Add the successful inference checkpoint (now published), recover the separate GPU/ROCm smoke-test transcript and model artifact hashes, and add missing checkpoints for Cloud Hermes post-reboot state and backup restore when evidence becomes available.
- Verify Strata API authentication, host-port exposure, all peers on claryel-private-ai, direct-IP versus Host-header behavior, and reasoning_content response handling before expanding the set of agent clients. For the API-key work, inspect write_private() call sites and preset serialization/mode, stat the exact active profile and parent directories, audit /data backup coverage, search direct-serve helpers for secret persistence, and trace every _authorized() route call site.
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


## 7. Cloud Hermes guest runtime discovery — 2026-11-07

New record: `logs/P2002/2026-11-07-cloud-hermes-guest-runtime-discovery.md`.

- Guest SSH is confirmed at `172.30.240.12` as `claryelvm`; hostname `claryel-cloud-vm`, Ubuntu 24.04.5.
- QEMU Guest Agent is not connected; SSH works.
- Owner reports existing Hermes installation, Grok authorization and response tests.
- Filtered guest diagnostics found no Hermes/Nous-named process or system/user service; user linger is disabled. This is **not** proof that Hermes is absent; installed-but-stopped binaries and other runtime managers have not been inspected.
- The public initial-install narrative and issue #8 do not contain the missing Hermes/Grok installation evidence. This is a provenance gap, not a reason to reinstall.
- No guest configuration or service was changed.

Next: grouped read-only inventory of binaries, package managers, containers, installation directories and alternative service/supervisor mechanisms. Never display credentials or secret values; do not reinstall or repeat Grok authorization.


### Recovered prior Cloud Hermes installation record

A targeted prior-conversation retrieval recovered Hermes v0.23.0 at `/home/claryelvm/.hermes/hermes-agent`, CLI `/home/claryelvm/.local/bin/hermes`, config paths `~/.hermes/config.yaml` and `~/.hermes/.env`, xAI OAuth configuration, selected model `grok-4.6`, base URL `https://api.x.ai/v1`, and a reported successful `hermes chat --oneshot` test. Classification: **PRIOR_CONVERSATION_RECOVERED**, not fresh shell evidence. No credential values or OAuth credential object identifiers are published.

No persistent service/container/autostart mechanism was documented in the recovered history. This reconciles the current no-process/no-matching-unit diagnostic: the CLI can exist and have passed a one-shot test without a persistent daemon. Do not reinstall or reauthorize. Inspect the existing CLI's supported gateway/daemon mode and configure its actual supported boot-start mechanism.
