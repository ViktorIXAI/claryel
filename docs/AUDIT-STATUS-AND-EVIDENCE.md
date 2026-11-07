# CLARYEL implementation evidence status

**As of 2026-11-07 (refreshed after owner-supplied direct host-to-container and full displayed Guard/iptables inspection).** This is a source-index and gap assessment, not live host telemetry or an independent security certification. Do not infer execution state from Git commit titles alone.

## Current scope

- **Log inventory:** the public logs root currently contains only logs/P2002/ with **16 standalone Markdown records** including the long initial-install narrative. See the [complete inventory and evidence-gap register](LOG-INVENTORY-AND-GAPS-2026-11-07.md).

- **Architecture:** v0.1 records the original Hermes/Strata-oriented isolation proposal. Owner has now requested vendor-neutral replaceable service roles, artifact admission/quarantine, fail-closed lifecycle and portable agent state; see [Architecture v0.2](ARCHITECTURE-V0.2-REPLACEABLE-PLATFORM.md). This is a design proposal, not deployed capability.
- **P2002:** laboratory host installation and incremental security/network validation are **in progress**. The owner supplied a post-reboot functional acceptance transcript for the restricted Strata proxy path and a subsequent model-download bring-up transcript; see the [P2002 network isolation and proxy acceptance log](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md) and [IQ3_S download bring-up log](../logs/P2002/2026-11-06-strata-iq3s-download-bringup.md). The network result is a bounded functional test, not a security certification.
- **Cloud Hermes:** host-side libvirt network `claryel-cloud-admin` and VM `claryel-cloud-hermes` have autostart enabled; the VM is running and the guest lease was observed at `172.30.240.12` in the owner-supplied snapshot. This is configuration/current-state evidence, **not a controlled host reboot test**. Inside the guest, the existing Hermes v0.23.0 CLI and xAI OAuth configuration were preserved; no reinstall or reauthorization occurred. Hermes PM was repaired offline from a local PEP 503 wheelhouse using original hash-verified `packaging 26.3` and `setuptools 84.1.0` artifacts. The Hermes Gateway user service is enabled and active, `Linger=yes`, and a controlled SIGKILL restart passed (PID 2969 → 3132; `NRestarts=1`). The owner explicitly does **not** want Telegram or any other messaging-platform integration; the Gateway's `No messaging platforms enabled` warning is expected. `hermes model` confirms primary `grok-4.6` via xAI Grok OAuth and credentials ✓, but the latest real `hermes chat --oneshot` request failed before model response because xAI OIDC discovery could not resolve DNS. The guest has no default route/upstream DNS and the observed Guard blocks forwarding on `virbr240`; no firewall/network change was made. Actual guest/host reboot persistence and a successful live Grok response remain NOT_TESTED / BLOCKED. See the [Cloud Hermes network/PM/Gateway checkpoint](../logs/P2002/2026-11-07-cloud-hermes-network-pm-repair-gateway.md), [host boot snapshot](../logs/P2002/2026-11-07-host-boot-state-snapshot.md), and [issue #10](https://github.com/ViktorIXAI/claryel/issues/10).
- **P2002 GPU / Strata:** owner-supplied output confirms Strata 0.2.11 readiness, Qwen3.8-Flash-Next IQ3_S loaded, and successful real inference from a disposable peer (48.6 generated tokens/s; 151.3 prompt tokens/s). Docker has no host port binding for 8080 and the host `ss` check found no listener on 8080, but a later direct host request to `172.31.250.2:8080/health` returned HTTP 200 via bridge `br-ffbceca294f9` (`172.31.250.1/24`). The inference network is `Internal=true`, IPv6 disabled; previous container egress tests were bounded to tested endpoints. The profile is root-owned mode `0600` and has neither `api_key` nor `allowed_hosts`; the service is unauthenticated. Strata native Bearer/`x-api-key` support for `/v1/*` is identified. The supplied `inet claryel_guard` and iptables rules cover `virbr240` and the separate proxy bridge `br-strata-eg` (`172.30.250.0/24`), not the inference bridge `br-ffbceca294f9` (`172.31.250.0/24`). Host `OUTPUT` policy is ACCEPT; `DOCKER-USER` is empty; `DOCKER-FORWARD` permits same-bridge forwarding. Guard enforcement for the inference bridge is not demonstrated. Public-WAN/LAN reachability and authentication coverage of non-`/v1/*` routes remain untested. The latest owner-supplied container inspect records immutable image digest `sha256:932700126d693662cbd11623555b9c451c7c678da138455b36bb05607f8d061b`, GPU devices `/dev/kfd` and `/dev/dri`, supplemental groups `993,44`, `unless-stopped` restart, 30s/8s/3-retry/2h healthcheck, writable named `/data` volume and read-only `claryel-strata-models` volume, no published ports, and no privileged mode. Container and image `Config.User` are empty, but the image entrypoint is `/usr/bin/tini -- /opt/strata/entrypoint.sh` with command `run`; owner-supplied in-container checks confirm active Python processes UID/GID 0:0 with supplementary groups 0,44,993. The profile is 0600 root:root; `/data` and `/models` mount roots are 0755 root:root. Docker-configured environment variable names were checked without values; `STRATA_API_KEY` is absent, consistent with `api_key=false`. The prior diagnostic's mount loop was malformed, but the subsequent corrected command successfully checked exact `/data`, profile and `/models` paths. Owner-supplied source inspection confirms the native API_KEY_FILE flow: the key is passed to strata launch via STRATA_LAUNCH_KEY, persisted by launch in the private preset and derived run config, and written by the entrypoint to /data/state/api-key under umask 077. The derived run-config writer uses a same-directory temporary file, chmod 0600 (best effort), and atomic replacement; launch_dir() resolves to paths.state_base()/launch. Preset save implementation/path/mode and /data backup coverage remain unverified. The latest filename-filtered find showed two runtime logs at 0600 root:root but did not match the active profile basename iq3_s-128k.json; it is not a recursive permission audit. The live container overrides the image entrypoint and directly runs strata serve --profile; the direct server accepts STRATA_API_KEY or profile api_key. Its _authorized() helper checks Bearer/x-api-key and returns 401 for bad/missing credentials, but route call-site coverage remains unproven. A file-reading wrapper remains only a candidate pending preset/backup review, direct-serve persistence/logging review, route audit and disposable dummy-key testing. No production secret was created. See [the inference/security checkpoint](../logs/P2002/2026-11-07-strata-private-api-inference-success.md), [log inventory](LOG-INVENTORY-AND-GAPS-2026-11-07.md), [issue #9](https://github.com/ViktorIXAI/claryel/issues/9) and [issue #16](https://github.com/ViktorIXAI/claryel/issues/16).
- **Hardware benchmark comparisons and production/customer use:** not established by this evidence index.

## Evidence and issues

| Area | Current public evidence | Limitation | Follow-up |
| --- | --- | --- | --- |
| Installation sequence | [P2002 journal](../logs/P2002/2026-11-06-initial-install.md) | Owner-provided observations, not live independent shell audit | [#3](https://github.com/ViktorIXAI/claryel/issues/3) |
| Pre-reboot guest-image checkpoint | [Backup verification](../logs/P2002/2026-11-06-pre-reboot-backup-verification.md) | Existing backup byte-identical to source; qcow2 check clean; guest shut off. Restore boot and independent backup recovery remain unverified. | [#3](https://github.com/ViktorIXAI/claryel/issues/3), [#5](https://github.com/ViktorIXAI/claryel/issues/5) |
| Strata network isolation and proxy persistence | [Post-reboot acceptance log](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md) | Owner-run positive/negative TCP and HTTPS CONNECT tests passed after one reboot. IPv6, DNS/UDP/QUIC, L2, SSRF edge cases, Guard loss/tampering, second reboot and service-failure recovery remain NOT_TESTED. | [#4](https://github.com/ViktorIXAI/claryel/issues/4), [#7](https://github.com/ViktorIXAI/claryel/issues/7) |
| Cloud Hermes guest network, PM repair and Gateway | [Cloud Hermes checkpoint](../logs/P2002/2026-11-07-cloud-hermes-network-pm-repair-gateway.md) | Offline package repair and controlled systemd SIGKILL restart passed. Guest has no default route/upstream DNS; Gateway has no messaging platform enabled; auxiliary provider warnings, end-to-end messaging and reboot persistence remain open. | [#10](https://github.com/ViktorIXAI/claryel/issues/10) |
| UFW / Guard integration | [UFW/Guard source](../logs/P2002/2026-11-06-ufw-guard-audit.md), [bridge inspection](../logs/P2002/2026-11-06-ufw-guard-boot-timing.md), [observed activation chronology](../logs/P2002/2026-11-06-systemd-activation-order.md), [effective dependency inspection](../logs/P2002/2026-11-06-effective-guard-dependencies.md), and [post-reboot proxy acceptance](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md) | One UFW reload preserved the live proxy exception. This does not prove every UFW operation or every firewall manager preserves the Guard. | [#1](https://github.com/ViktorIXAI/claryel/issues/1), [#4](https://github.com/ViktorIXAI/claryel/issues/4), [#7](https://github.com/ViktorIXAI/claryel/issues/7) |
| Backup integrity | [Pre-reboot backup records](../logs/P2002/2026-11-06-pre-reboot-backup-inventory.md) and [network acceptance backup audit](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md) | Guard deployment backup checksums passed. Proxy persistence backup contains iptables/nftables/Squid snapshots but lacks a SHA256SUMS manifest; its manifest is still outstanding. | [#2](https://github.com/ViktorIXAI/claryel/issues/2), [#5](https://github.com/ViktorIXAI/claryel/issues/5) |
| RX 9070 / Strata / Qwen model | [Network acceptance](../logs/P2002/2026-11-06-strata-network-isolation-acceptance.md); [IQ3_S download bring-up](../logs/P2002/2026-11-06-strata-iq3s-download-bringup.md); [verified inference/security checkpoint](../logs/P2002/2026-11-07-strata-private-api-inference-success.md) | Host-to-container HTTP 200 is confirmed through the Docker bridge despite no published host port/listener. Profile key status is now known: no `api_key` or `allowed_hosts` field; API remains unauthenticated. Displayed Guard/iptables rules do not cover the inference bridge; host OUTPUT is ACCEPT and DOCKER-USER is empty. Public-WAN/LAN reachability and non-`/v1/*` route authentication remain NOT_TESTED. Standalone ROCm/HIP smoke-test transcript also remains missing. | [#9](https://github.com/ViktorIXAI/claryel/issues/9), [#15](https://github.com/ViktorIXAI/claryel/issues/15), [#16](https://github.com/ViktorIXAI/claryel/issues/16) |
| Publication safety | [Two-ledger policy](../policies/PUBLICATION-SECURITY.md); [public P2002 architecture/client identity standard](../policies/PUBLIC-ARCHITECTURE-AND-CLIENT-IDENTITY.md) | Owner now explicitly authorizes public disclosure of P2002 non-secret architecture and lab identifiers. Mandatory prepublication enforcement and exact-byte scanning are still not verified. | [#1](https://github.com/ViktorIXAI/claryel/issues/1) |
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


## New architecture/security analysis backlog

- [#11 — artifact supply-chain admission](https://github.com/ViktorIXAI/claryel/issues/11)
- [#12 — fail-closed lifecycle and no startup gaps](https://github.com/ViktorIXAI/claryel/issues/12)
- [#13 — vendor-neutral replaceable service roles and APIs](https://github.com/ViktorIXAI/claryel/issues/13)
- [#14 — portable agent state and migration](https://github.com/ViktorIXAI/claryel/issues/14)
- [#15 — complete log inventory and missing GPU/Strata evidence](https://github.com/ViktorIXAI/claryel/issues/15)
- [#16 — authenticate Strata API and verify inference-network exposure](https://github.com/ViktorIXAI/claryel/issues/16)
- [P2002 Strata API authentication change and rollback plan](P2002-STRATA-API-AUTH-CHANGE-PLAN.md) — proposed only; no live change authorized/performed
- [#17 — generate unique deployment identity and network namespace for every client Box](https://github.com/ViktorIXAI/claryel/issues/17)


## Current execution priority — owner direction 2026-11-07

The owner has reaffirmed that the immediate objective is to finish installing and starting the complete working P2002 stack. The successful owner-supplied Strata/Qwen inference is the first-inference milestone; the next operational priority is to resume Cloud Hermes startup, install/start Hermes, connect it to the existing local Strata endpoint over the current private lab network using synthetic data, and demonstrate one end-to-end request/reply.

Do not block this integration sprint on deep API-auth research, wrapper development, exhaustive route audits, standalone GPU evidence recovery, broad benchmarking or non-blocking security test suites. These remain open follow-up work, especially issues #9 and #16. Preserve the current Guard/firewall, private network and no-published-port configuration; do not expose the inference endpoint outside the lab or use real private/customer data before the authentication/Security Gateway boundary is completed. Record each actual installation/change step and its evidence. This is a priority change, not a security approval or authorization to weaken existing controls.


## Owner correction — Cloud Hermes guest agent already installed (2026-11-07)

The owner reports that Hermes is **already installed inside** the existing `claryel-cloud-hermes` VM, is connected to Grok with provider authorization completed, and has passed response tests. This supersedes the earlier forward-looking wording above that said to install/start Hermes. **Do not reinstall Hermes or repeat provider authorization.**

Evidence classification is **OWNER_REPORTED**: the public repository snapshot reviewed at this checkpoint did not yet contain the original guest installation transcript, exact Hermes version/service-unit name, Grok authorization evidence, or response-test output. The new append-only correction is recorded in [the guest-agent correction checkpoint](../logs/P2002/2026-11-07-cloud-hermes-agent-existing-correction.md).

The host-side libvirt network and VM autostart flags have now been enabled and recorded in [the autostart checkpoint](../logs/P2002/2026-11-07-cloud-hermes-autostart.md). This is separate from guest Hermes service autostart, which remains to be inspected.

**Next operational step:** inspect the already-installed guest Hermes service and enable its existing boot-start mechanism if needed; then proceed to connect this existing agent to the already-working local inference API. Do not perform a reinstall. Preserve the current Guard/firewall and network boundaries.


## Cloud Hermes guest runtime discovery — owner-supplied output 2026-11-07

The owner successfully SSH-connected through the existing ASUS jump host to guest `claryelvm@172.30.240.12` (hostname `claryel-cloud-vm`, Ubuntu 24.04.5, kernel 6.8.0-147-generic). DHCP lease identifies MAC `52:54:00:40:bd:81` and the same guest IP. QEMU Guest Agent is not connected, but SSH access works.

The owner reports that Hermes is already installed, connected/authorized with Grok, and has passed response tests. Do not reinstall it or repeat authorization. However, the current guest diagnostics returned no Hermes/Nous-matching system or user service units and no matching process; `loginctl show-user claryelvm -p Linger` returned `Linger=no`. These filtered checks do not prove the software is absent: its executable may be installed but stopped, use another name, or be managed by a container, supervisor, cron, tmux or custom launcher. No changes were made.

The new read-only checkpoint is [Cloud Hermes guest runtime discovery](../logs/P2002/2026-11-07-cloud-hermes-guest-runtime-discovery.md). **Next:** one grouped inventory of existing guest installation footprints and runtime managers, with secret values excluded. Then enable the actual existing startup mechanism if needed and continue local-inference integration.


## Recovered existing Cloud Hermes installation and Grok test (prior conversation, 2026-11-06)

A targeted prior-conversation retrieval recovered historical setup facts not yet present in the public logs: Hermes v0.23.0 was installed at `/home/claryelvm/.hermes/hermes-agent`, with CLI `/home/claryelvm/.local/bin/hermes`; configuration files were `~/.hermes/config.yaml` and `~/.hermes/.env`. xAI OAuth had been configured, `grok-4.6` selected, API base URL `https://api.x.ai/v1`, and `hermes chat --oneshot` reportedly succeeded. This is classified as **PRIOR_CONVERSATION_RECOVERED**, not a fresh terminal transcript; no token or credential object identifier is published.

The retrieved history did not document a persistent service/container or autostart mechanism. The current owner-supplied guest diagnostics show no matching process or Hermes/Nous-named system/user unit and `Linger=no`, which is consistent with an installed CLI that was tested in one-shot mode but not configured as a persistent boot-time agent.

**Do not reinstall Hermes or repeat Grok authorization.** Next inspect the existing CLI's help for its persistent/gateway mode and establish a boot-start service for the existing installation. See [the updated guest runtime discovery checkpoint](../logs/P2002/2026-11-07-cloud-hermes-guest-runtime-discovery.md).

## Cloud Hermes guest package repair and Gateway runtime — owner-supplied output 2026-11-07

The later guest transcript supersedes the earlier pre-install snapshot that showed no matching Hermes process/unit and `Linger=no`:

- Guest DNS/default-route diagnosis confirmed `172.30.240.12/24` has no default route and no upstream DNS scope. Host Guard drops forwarding for `virbr240`; the Squid proxy is bound to the separate `br-strata-eg` network.
- The network isolation policy was not modified. Dependency repair was completed offline after replacing reconstructed packaging/setuptools wheels with original PyPI artifacts matching the guest lock-file hashes.
- `hermes pm repair` completed successfully and `hermes pm doctor` reported no corrupt UV cache entries.
- `hermes gateway install --start-now --start-on-login` installed and started the user service; `loginctl enable-linger` resulted in `Linger=yes`.
- A deliberate SIGKILL test caused systemd to restart the Gateway: PID 2969 → 3132; `NRestarts=1`; service active/running.
- The Gateway journal says `No messaging platforms enabled`. It also reports missing auxiliary xAI OAuth and Nous Portal credentials. These warnings do not independently prove the primary Grok session is disconnected.
- No actual logout, VM/host reboot, inbound/outbound message test, or network-rule change is evidenced.

See [the complete append-only technical checkpoint](../logs/P2002/2026-11-07-cloud-hermes-network-pm-repair-gateway.md).

**Next:** inspect the installed CLI's supported Gateway/messaging setup, identify the intended messaging platform, configure it without exposing credentials, then run a controlled inbound/outbound test. Keep the guest's current network isolation; do not add NAT/default routing merely to clear the PyPI DNS failure.



## Execution triage — 2026-11-07

The next installation milestone is a working Cloud Hermes Gateway request to its already-configured Grok provider through a narrowly controlled egress path. The current DNS failure is not evidence of missing OAuth credentials. Do not add a guest default route, NAT, broad forwarding or general Internet access.

A separate proposed xAI proxy path is tracked in [issue #19](https://github.com/ViktorIXAI/claryel/issues/19); the prioritized operator sequence and deferral list are in [Execution triage](EXECUTION-TRIAGE-2026-11-07.md). No host/network mutation has been made by this triage.

**Recovery gate:** the previously verified qcow2 backup predates the later Hermes PM/Gateway changes. Its earlier byte-integrity result must not be represented as a current restore point for the guest. Confirm current backup metadata and recovery scope before any host reboot.

**Architecture gate:** issue #8's older direct Hermes-to-Strata instruction conflicts with Architecture v0.1/v0.2, which requires cross-zone exchanges through an independent Security Gateway. Strata currently reports `api_key=false`. Keep the immediate Cloud Hermes demo Grok-only; do not attach Cloud Hermes directly to Strata or introduce private data until the architecture conflict is resolved and the Security Gateway/API boundary is implemented and reviewed.

This triage is based on repository evidence and owner-supplied terminal transcripts, not live host telemetry. No reboot, firewall, Guard, routing, DNS, Docker, VM, disk or service changes were performed.
