# CLARYEL

**Status: architecture proposal with laboratory implementation in progress on P2002. Not production-ready. No security certification or legal compliance is claimed.**

CLARYEL is an experimental, offline-first local AI system intended for privacy-sensitive environments. Development is greenfield, incremental, evidence-led and subject to explicit owner approval at every deployment gate. Previous repositories are untrusted reference material, not code/configuration to import automatically.

## Open security research: external and internal threats

**We publish our experimental architecture and detailed test evidence to invite independent scrutiny by security professionals and the technical community. Our intended customer deployments will preserve the same foundational security principles while keeping sensitive, customer-specific operational configurations confidential. Our long-term objective is independent, confidential professional security assessment and, where applicable, formal certification.**

**Every container and every AI workload is treated as a potential source of risk—not only a possible target of external attacks.** Models, agent frameworks, inference engines, plugins, skills, dependencies, installers, and updates may contain vulnerabilities, malicious behavior, or simply unexpected functionality. Even a legitimate model or agent can inadvertently disclose data through outputs, tool calls, network requests, logs, caches, or shared state. Containerization alone is not a security guarantee.

**CLARYEL is designed to contain the consequences of compromise or failure inside its own components**, using independently enforced least privilege, default-deny networking, separated trust zones, strict data access, controlled cross-zone operations, quarantined updates, and fail-closed lifecycle controls. No AI model or agent should be able to authorize its own privileges or bypass those boundaries. These are security objectives under test, **not claims that all controls have already been implemented or independently validated**.

Read the complete [Public Security Review and Production Security Strategy](docs/PUBLIC-SECURITY-REVIEW-STRATEGY.md), including internal workload threats, responsible disclosure, customer-specific protections, and independent assessment goals.

## Foundational rules
- **Assume compromise:** no software component, model, agent, plugin, device, update or external response is inherently trusted. Minimize and verify the unavoidable trusted computing base (firmware, host, hypervisor, isolation and keys).
- **Default deny:** enforce least privilege independently of model instructions at the OS/hypervisor/network/storage layers.
- **Offline-first:** supported private AI, private documents and local Home Assistant functions remain usable with WAN physically disconnected; cloud functions fail closed.
- **Two untrusted agent roles:** a private agent in a restricted zone and an external/cloud-capable agent in an isolated zone. The current implementation may use Hermes, but the framework is replaceable. **No direct agent-to-agent connection.** An independent **Security GTW** is the sole permitted communication intermediary.
- **Strict first release:** no private documents, excerpts, embeddings, personal data or confidential context may pass from private to public/cloud zones. A configurable hybrid option is future work, disabled by default and requires legal and security review.
- **Inference only:** the local inference service and selected model have no document-store mount, shell/MCP/tool execution privilege or internet access. Current experiments use Strata and Qwen, but both runtime and model are replaceable. One physical GPU is a cost target, **not** proof that a shared inference process safely isolates users, context, cache and logs.
- Home Assistant, identity, backups, updates, telemetry and administration receive separate restricted security boundaries.
- Every update enters quarantine; require provenance, staged testing, explicit approval, rollout and rollback.
- **AI-assisted and fallible by design:** ChatGPT, Grok, local Hermes-based AI and potentially other agents may contribute and publish over time. Human contributors and all AI systems can make mistakes; preserve reviewable history and analyze errors in both reproduced and newly designed solutions.
- Publish only in **English**, with secret/PII/operational-information review before any public commit.

## Current implementation evidence

- [Current audit status, evidence index and unresolved gaps](docs/AUDIT-STATUS-AND-EVIDENCE.md)
- [P2002 log inventory and missing-evidence register (2026-11-07)](docs/LOG-INVENTORY-AND-GAPS-2026-11-07.md)
- [Forward-looking audit checkpoint template](templates/AUDIT-CHECKPOINT.md)
- [Tracking issues: security publishing gate, private evidence ledger and reproducibility](https://github.com/ViktorIXAI/claryel/issues)

The P2002 installation journal contains **operator-reported** findings; Git publication does not itself independently verify execution. Pre-publication scanning enforcement and a protected private evidence ledger are **not yet demonstrated**.

## Project map
- [Architecture v0.1 — original trust boundaries](docs/ARCHITECTURE.md)
- [Architecture v0.2 — vendor-neutral replaceable platform, artifact admission and fail-closed lifecycle](docs/ARCHITECTURE-V0.2-REPLACEABLE-PLATFORM.md)
- [Owner-controlled implementation roadmap](docs/ROADMAP.md)
- [P2002 and P1005 hardware testbeds](docs/HARDWARE-TESTBEDS.md)
- [Storage, encrypted keys, state indicators and deferred backup plan](docs/STORAGE-KEYS-AND-BACKUP-PLAN.md)
- [Persistent 64 GB recovery USB standard](docs/PERSISTENT-RECOVERY-USB.md)
- [EU / Italian legal design baseline](docs/LEGAL-BASELINE.md)
- [Public English-only policy](policies/PUBLICATION-LANGUAGE.md)
- [Mandatory per-chat, AI-agent and per-Hermes-instance contributor provenance](policies/CONTRIBUTOR-PROVENANCE.md)
- [Two-ledger secure publication and reproducibility policy](policies/PUBLICATION-SECURITY.md)
- [Public P2002 architecture disclosure and per-client generated identity standard](policies/PUBLIC-ARCHITECTURE-AND-CLIENT-IDENTITY.md)
- [Security disclosure and incident policy](SECURITY.md)
- [AI-assisted authorship, fallibility and historical integrity policy](policies/AI-DEVELOPMENT-AND-HISTORY.md)
- [Update quarantine policy](policies/UPDATE-QUARANTINE.md)
- [Artifact supply-chain admission — issue #11](https://github.com/ViktorIXAI/claryel/issues/11)
- [Fail-closed boot/restart/service replacement — issue #12](https://github.com/ViktorIXAI/claryel/issues/12)
- [Vendor-neutral replaceable service roles — issue #13](https://github.com/ViktorIXAI/claryel/issues/13)
- [Portable agent state and migration — issue #14](https://github.com/ViktorIXAI/claryel/issues/14)
- [Synthetic security test plan](tests/SECURITY-TEST-PLAN.md)

**Security principle:** controls are enforced by deterministic systems outside AI agents. Never treat passing tests as proof of absolute safety. Never expose actual customer material or infrastructure secrets in this public repository.

- [Mandatory installation audit logging](policies/INSTALLATION-AUDIT-LOGGING.md)
