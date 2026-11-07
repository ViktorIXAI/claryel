# CLARYEL P2002 execution triage — 2026-11-07

**Purpose:** prioritize the next installation actions without allowing deep assurance work to block the first useful running system. This is a repository-based triage, not live host telemetry.

**Repository snapshot:** default branch `main`; no open pull requests were returned by the current GitHub query. The repository had 18 open issues before this triage; issue #19 was created to track the narrow xAI egress blocker. The repository is public by owner direction for the P2002 reference lab. No credentials or token values are included here.

## Executive decision

The next milestone is **a working Cloud Hermes Gateway that can make a real request to its already-configured Grok provider, using narrowly controlled egress**. Do not begin by redesigning the firewall, adding a default route, opening NAT or connecting more workloads.

The immediate blocker is guest DNS/egress, not a missing Hermes installation or missing primary Grok OAuth credentials.

The current owner-supplied state is not fresh enough to justify a reboot or network mutation. First capture a short read-only snapshot and confirm the current recovery point.

## P0 — do now, in this order

### 1. Refresh current state (read-only)

On P2002:
- confirm CLARYEL Guard service and exact policy verifier;
- confirm libvirt network and VM state/autostart;
- confirm guest DHCP lease and current disk attachment;
- confirm Docker/Strata remains running and the inference network peers/port bindings have not changed;
- inventory current VM backup files by name, size and timestamp only.

Inside the Cloud Hermes guest:
- confirm Gateway user unit enabled and active;
- confirm user linger;
- confirm current model/provider selection without reading or printing auth files;
- confirm route and resolver state;
- verify that no unexpected messaging service is configured (owner explicitly does not want messaging platforms).

This is a diagnostic checkpoint only. No reboot, restart, route, DNS, firewall, Guard, Docker or VM changes.

### 2. Restore xAI access through a narrow proxy — proposed, not applied

The installed Hermes source commit `2bd378ed1b8d6bf2192cb26e0a55ff3091e6bd19` defines:
- OIDC issuer/discovery: `https://auth.x.ai`;
- device-code endpoint: `https://auth.x.ai/oauth2/device/code`;
- default inference base: `https://api.x.ai/v1`.

The live discovery document currently advertises authorization, device-code, token, userinfo, revocation and JWKS endpoints on `auth.x.ai`.

Existing proxy `172.30.250.1:3128` belongs to the Strata egress bridge and is currently restricted to the model-download client and Hugging Face hosts. It must not be repurposed or broadened casually.

Issue [#19 — narrow xAI connectivity](https://github.com/ViktorIXAI/claryel/issues/19) tracks a proposed separate proxy listener for the Cloud Hermes guest:
- exact source `172.30.240.12/32`;
- proposed host-side listener `172.30.240.1:3129`;
- only `auth.x.ai:443` and `api.x.ai:443` via HTTPS CONNECT;
- deny all other hosts, private/reserved destinations and other guest sources;
- one narrowly scoped Guard INPUT exception if read-only inspection confirms the current chain order;
- keep no guest default route, no NAT and no broad FORWARD permission.

Before any mutation, inspect current Squid config, host resolver, Guard rules and UFW behavior; then present exact commands and rollback for owner approval. The proxy change is not authorized by this triage.

### 3. Prove one real Grok request

After the approved proxy path is installed:
- verify xAI OIDC discovery;
- run one synthetic one-shot request through the existing CLI;
- confirm the Gateway remains active;
- prove a non-allowlisted destination and private/LAN destination are denied.

No need to reauthorize, reinstall Hermes, configure Telegram/Discord, or repeat PM repair unless new evidence shows those components have regressed.

### 4. Confirm recovery before reboot

The pre-reboot qcow2 backup previously passed byte-identity and structural checks, but it predates later Hermes PM/Gateway changes. It must not be treated as a verified restore point for the current guest state without checking its timestamp/content scope.

Inventory the current VM disk and backup metadata; design a current recoverable checkpoint. Do not reboot P2002 until the owner reviews the recovery plan and explicitly approves the controlled reboot. Then test guest Gateway persistence and host VM/network autostart separately and record actual results.

## P1 — next after the Grok demo

1. **Issue #16 — Strata API authentication and exposure.** Keep the current Strata endpoint private and do not attach any agent or real/private data until its authentication and route coverage are proven. Existing evidence says `api_key=false`; the endpoint has no published host port, but a peer on the private Docker network can use it.
2. **Issue #12 — fail-closed lifecycle.** Resume the full boot/restart/crash/service-replacement analysis after installation is assembled. No live fault injection now.
3. **Issue #11 — artifact supply chain.** Build the intake/quarantine/admission process before promoting future untrusted artifacts or customer deployments.
4. **Issue #9 — RX 9070/ROCm evidence.** Model inference is already owner-reported successful (48.6 generated tok/s); recover the separate raw gfx1201/ROCm/HIP smoke-test transcript. This is an evidence gap, not a current GPU failure.
5. **Issue #15 — full installation timeline.** Continue reconciling missing transcripts, provenance and evidence references without blocking the current functional milestone.

## Important architecture conflict — do not silently bypass

Issue #8 contains an older instruction to connect Cloud Hermes directly to the local Strata endpoint over the private network. The current Architecture v0.1 and v0.2 documents require all private/external cross-zone exchanges to pass through the independent Security Gateway; agents must not connect directly. Strata is currently unauthenticated, and the Security Gateway is not demonstrated as deployed.

Therefore:
- the immediate Cloud Hermes milestone is Grok-only through narrow xAI egress;
- do not attach Cloud Hermes directly to Strata or bridge the two networks;
- before Hermes-to-local-Strata integration, reconcile the issue #8 instruction with the current architecture and implement/review the Security Gateway and Strata API authentication boundary;
- use synthetic data only until those boundaries are accepted.

This is a safety-preserving interpretation of the current repository documents, not a claim that the Security Gateway already exists.

## Deferred — do not spend today's installation window here

- #1 mandatory publication gate: policy is documented; full enforcement remains unproven. Continue manual reviewed publication under the owner's standing instruction; implement the gate before production/client operation.
- #2 private evidence ledger: not deployed. Design/test with synthetic data after the working milestone; do not store secrets in public Git.
- #3 richer checkpoint schema and machine-readable evidence links.
- #4 broad reproducible firewall/Docker/KVM test fixtures.
- #6 enforced per-writer identity and attribution.
- #7 Guard table-loss and live failure-injection tests (explicitly deferred).
- #13 vendor-neutral service/API design beyond the current architecture document.
- #14 agent skill/memory/config portability.
- #17 unique customer deployment identity and generated service/network namespaces.
- #18 ripgrep/PATH discrepancy (P3, non-blocking).

These items remain open; deferral is not closure or a security pass. No customer data, production deployment or broad external access is authorized by this triage.

## Latest read-only preflight — host/guest snapshots completed 2026-11-07

Owner supplied the P2002 host snapshot at 10:03:40 UTC and Cloud Hermes guest snapshot at 10:05:06 UTC, followed by additional host-side read-only transcripts. Full sanitized record: [xAI egress preflight log](../logs/P2002/2026-11-07-xai-egress-preflight.md).

Confirmed:
- Guest Hermes v0.23.0 is up to date, `grok-4.6` selected, primary xAI OAuth credentials present, Gateway enabled/active, Linger=yes, and guest has no default route or DNS scope (from the earlier guest-side snapshot).
- Host xAI HTTPS works: OIDC discovery on `auth.x.ai` returns HTTP 200 with TLS verification 0; unauthenticated `api.x.ai/v1/models` returns HTTP 401 with TLS verification 0. This confirms transport/TLS only, not authenticated inference.
- Pinned upstream Hermes source (`2bd378ed1b8d6bf2192cb26e0a55ff3091e6bd19`, `hermes_cli/auth_xai.py`) uses default `httpx.get`/`httpx.Client` calls for xAI OIDC discovery, refresh and device-code flow without explicitly disabling environment proxy handling. This suggests `HTTPS_PROXY` should work for those OAuth requests under standard HTTPX defaults, but the installed guest source/runtime and the inference transport remain unverified. See [pinned source](https://github.com/NousResearch/hermes-agent/blob/2bd378ed1b8d6bf2192cb26e0a55ff3091e6bd19/hermes_cli/auth_xai.py).
- The complete canonical `/etc/claryel/firewall.nft` is now captured. It contains exactly 12 explicit rules across the two Guard chains. Its current `guest_input` chain drops all new `virbr240` input after established/related and DHCP exceptions; no Cloud Hermes TCP/3129 exception exists.
- **Guard baseline discrepancy:** live SHA-256 for `firewall.nft` and `claryel-guard-verify` do not match the dated deployment manifest; `claryel-guard-load` does match. This is unresolved baseline drift, not proof of malicious change or runtime failure. A prior 10:05 snapshot reported verifier PASS, but the latest transcript did not rerun it. Capture the full current verifier, rerun it, compare the live nft table to the canonical file, and reconcile the dated backup before modifying either Guard artifact.
- UFW/iptables earlier showed IPv4 INPUT policy DROP and no current TCP/3129 allow. A matching exact UFW INPUT rule will be needed alongside the Guard exception; no broad libvirt rule, FORWARD rule, NAT or guest default route is indicated.
- Full existing Squid config is now captured. It has one listener `172.30.250.1:3128`, Strata source `172.30.250.10/32`, exact Hugging Face/GitHub domains, HTTPS CONNECT/443 restrictions, private/reserved destination denial and default deny. Parser output showed no parse error, but its exit code was not captured. The listener listing showed only 3128.
- Issue #19 explicitly says to keep the existing Strata/Hugging Face proxy ACL and listener unchanged. Therefore prefer a **separate Squid instance** for Cloud Hermes, with independent config, PID, logs and runtime/cache paths, rather than modifying the existing Squid configuration or adding a second listener to it.
- Docker state is confirmed: `claryel-inference` is running/healthy on `claryel/strata:0.2.11-rocm-gfx120x`, attached only to internal network `claryel-private-ai` (`internal=true`), address `172.31.250.2`, with no published host port.
- Live VM block metadata: disk target `vda`, capacity 40.000 GiB, allocation/physical 5.300 GiB. This is not a backup or image consistency check. The active-image lock warning remains expected; do not bypass it.
- Existing 2026-11-06 backup artifacts predate the 2026-11-07 Hermes PM/Gateway repair and are not a current recovery point. Backup filesystem had 223 GB free in the earlier host snapshot.
- The latest Gateway unit and Hermes source commands were run on host `p2002`, not inside the guest. Empty host user-unit paths and a missing host-side source directory do not establish guest Gateway state or proxy support. Guest-side metadata remains NOT_CAPTURED.

### Next operator action — final read-only reconciliation

1. Capture the full current `/usr/local/sbin/claryel-guard-verify` and `claryel-guard-load`, rerun the verifier, and capture `nft list table inet claryel_guard`. Compare the live ruleset, canonical file and verifier's exact rule array.
2. Inspect the dated Guard deployment backup directory and manifest to identify whether the two differing hashes reflect an intentional later revision or unreviewed drift. Do not restore from the old manifest or overwrite either live file.
3. Run Gateway unit/source inspection **inside the Cloud Hermes guest**. Capture unit fragment/drop-in paths and only proxy variable names as SET/UNSET; never print environment values, OAuth tokens or credentials. Confirm the exact installed HTTP/OIDC transport honors proxy settings.
4. Inspect Squid package/service capabilities for a second independent instance and its non-overlapping PID, log, runtime and cache paths. Keep the current Strata config/listener unchanged.
5. After the exact patch and rollback are reviewed, obtain explicit owner approval before any host/guest mutation or service reload. A current consistent guest backup and approved window remain required before reboot.

### Proposed change after reconciliation — NOT IMPLEMENTED

- Independent Squid instance bound only to `172.30.240.1:3129`; source fixed to `172.30.240.12/32`; CONNECT TCP/443 only; exact destinations `auth.x.ai` and `api.x.ai`; private/reserved destination denial and default deny. Its config, PID, logs and runtime/cache state must not overlap with the Strata Squid instance.
- One exact Guard INPUT exception for `virbr240`, source `172.30.240.12`, destination `172.30.240.1`, TCP/3129, before the broad guest input DROP; update canonical rules and verifier together only after baseline reconciliation.
- One exact UFW INPUT allow for the same source/interface/destination/port.
- Configure only the guest Hermes Gateway service to use the proxy after verifying the installed client/OIDC code honors it; no credentials in the service override.
- Test positive access to both xAI hosts and negative access to other public hosts and private/LAN destinations, then OIDC discovery and one authenticated Grok request.

No host or guest mutation, firewall/Squid reload, Docker restart, Gateway restart, disk copy/snapshot or reboot has been performed or authorized. Preserve the guest's lack of default route and the existing Strata private network.

## Evidence and provenance

- Repository main branch and issue/commit inventory inspected on 2026-11-07.
- Owner-supplied terminal evidence from Cloud Hermes and Strata logs; not live telemetry.
- Pinned Hermes Agent source at `2bd378ed1b8d6bf2192cb26e0a55ff3091e6bd19`.
- Live xAI OIDC discovery document: `https://auth.x.ai/.well-known/openid-configuration`.
- Writer-instance: `CHATGPT-6-TRIAGE-01`.
- Writer-role: `INSTALLATION_TRIAGE_AND_REPOSITORY_MAINTENANCE`.
- Executed-by: repository review and documentation only; no P2002 host command executed.
- Approved-by: owner's request to continue triage; no host/network mutation or reboot approval implied.
- Secrets: no credentials, token values, private keys or customer data included.
