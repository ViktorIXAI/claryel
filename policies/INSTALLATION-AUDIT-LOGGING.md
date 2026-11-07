# Mandatory Public Installation and Configuration Audit Logging

**Status: owner-directed policy — effective 2026-11-06 until explicitly revoked by the owner.**

> **Standing owner direction — 2026-11-07:** The P2002 lab's complete non-secret architecture and operational identifiers (including actual internal IPs, container/network/service names, ports, routes, firewall rules and test output) are to be published for public security review. This supersedes the expired temporary window for that lab scope. Credentials, private keys, customer data and unrelated personal/network identifiers remain excluded. Apply [the public architecture and client identity standard](PUBLIC-ARCHITECTURE-AND-CLIENT-IDENTITY.md). Publication authorization does not authorize host changes.

## Scope
Applies to the current P2002 system and every subsequent CLARYEL Box throughout installation, configuration, experiments, debugging, rollback, updates, recovery and security checks. The default is **publicly documented, independently auditable work**, not undocumented "successful" deployment. Public prose, scripts, commits, issues and technical reports are **in English**.

All public writers must follow the [contributor provenance policy](CONTRIBUTOR-PROVENANCE.md): distinct chats, models, automation processes and **each Hermes instance** (including public/private and same-role agents on different hosts) declare persistent writer aliases, roles and non-sensitive deployment aliases even when using the same GitHub login. This policy is documentary until the publication gate implements verified enforcement.

## Every consequential step must record
1. Box identifier, date/time with timezone when known, stage and initiator/actor (human, ChatGPT, Hermes or other model), with attribution only when evidenced.
2. Objective, rationale, security/privacy implications, preconditions and exact execution context (target machine, user, shell, environment).
3. Proposed commands or configurations **before execution** and the approval/execution status; distinguish suggested, executed, observed and independently verified.
4. Exit status, observed outputs or safely sanitized excerpts, version numbers, configuration digests, checks and deviations.
5. Errors, incorrect-terminal human actions, failed assumptions, AI errors, aborted work, corrections, negative evidence and discrepancies. Preserve historical mistakes and audit trail; do not silently rewrite outcomes.
6. Verification evidence, pass/fail/unknown criteria, residual risks, undo/recovery steps, and next action. Use timestamps, issue/commit references, and test identifiers to support external reproductions.

## Public safety boundaries
- **Never disclose** real passwords, passphrases, private SSH keys, API tokens, active credentials, private customer data or unrelated personal identifiers. For the P2002 reference lab, the owner explicitly authorizes publication of non-secret network topology, actual internal IPs, container/network/service names, access rules and security findings. For customer and third-party deployments, do not publish live addresses, hostnames, MACs, serials, key fingerprints or deployment manifests; use generated identifiers and synthetic examples.
- Review terminal output and scripts **before publication**, not after. Preserve P2002 non-secret operational details where relevant to the public architecture; redact secrets, customer/personal data and unrelated identifiers. If classification is uncertain, hold the artifact and publish a summary with an explicit withholding reason. Do not use public git history as a secrets vault or rely on deleting the latest version to remove an exposed secret.
- Preserve a **private, protected evidence trail** for sensitive details when appropriate; publish hashes or sanitized reproductions that allow independent review without exposing keys or internals.
- Unencrypted initial P2002 root remains **test-only** with synthetic data. No client data or production credentials until required controls and owner gates.
- Logs do not grant other AIs remote execution or administrative access. Independent reviewers may comment or propose changes; changes remain owner-approved and checked.

## Recommended repository structure
- `logs/<BOX-ID>/YYYY-MM-DD-<event>.md` — immutable-ish dated, append-oriented factual event and outcomes.
- `tests/` — reproducible test cases, expected results, evidence summaries and failure modes.
- `docs/` — current deployment and architectural state; link revisions and corresponding event logs.
- `policies/` — applicable controls and publication rules.
- Future automatic local audit capture and GitHub forwarding must undergo a separate least-privilege design review. **This document does not claim automatic host logging has been installed.**

## Execution protocol to minimize human and AI mistakes
- Explicit target label on every command: **ASUS workstation** or **P2002 host**, etc.
- Read-only preflight; one small change; post-change validation; independently tested recovery/rollback when practical.
- Connectivity-sensitive changes must preserve an existing console/SSH route and a rollback procedure before activation.
- Destructive disk operations, encryption migrations, firewall locks, secrets operations and public data transfers require explicit checks and owner-controlled authorization.
- If a tool call fails, a result is absent, or a host has not been checked, record **not verified** rather than claiming success.
- No model, including ChatGPT, is infallible; the user instruction to follow ChatGPT-proposed commands is a demand for **accountable, reviewable procedures**, not a waiver of checks.

## Current P2002 checkpoint
- Initial installation, dual-GPU display issue, user error creating an SSH key on the target and subsequent verified removal, ASUS key-only SSH access, KVM preflight, and router/DHCP observations were previously recorded in the dated P2002 log.
- UFW was previously **inactive** and an explicit allow rule for the currently connected administrative workstation was **added and verified with `ufw show added`**. **No `ufw enable` execution or successful second SSH test has been reported at the time of this policy.** Do not mark the firewall as enabled.
- Recovery USB, secrets broker, backup/restore, and full hardening are planned but not yet demonstrated.

## Revocation and change control
Continue this standard for **all boxes until the owner explicitly instructs otherwise**. Changes to policy require dated, public, attributable revisions, except safety-mandated redaction and handling of unauthorized disclosure.
