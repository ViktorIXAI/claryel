# CLARYEL

**Status: architecture proposal / pre-implementation. No deployment, security certification, or legal compliance is claimed.**

CLARYEL is an experimental, offline-first local AI system intended for privacy-sensitive environments. Development is greenfield, incremental, evidence-led and subject to explicit owner approval at every deployment gate. Previous repositories are untrusted reference material, not code/configuration to import automatically.

## Foundational rules
- **Assume compromise:** no software component, model, agent, plugin, device, update or external response is inherently trusted. Minimize and verify the unavoidable trusted computing base (firmware, host, hypervisor, isolation and keys).
- **Default deny:** enforce least privilege independently of model instructions at the OS/hypervisor/network/storage layers.
- **Offline-first:** supported private AI, private documents and local Home Assistant functions remain usable with WAN physically disconnected; cloud functions fail closed.
- **Two untrusted Hermes agents:** private Hermes in a restricted zone and cloud-capable Hermes in an isolated VM. **No direct agent-to-agent connection.** An independent **Security GTW** is the sole permitted communication intermediary.
- **Strict first release:** no private documents, excerpts, embeddings, personal data or confidential context may pass from private to public/cloud zones. A configurable hybrid option is future work, disabled by default and requires legal and security review.
- **Inference only:** local Strata + LLM has no document-store mount, shell/MCP/tool execution privilege or internet access. One physical GPU is a cost target, **not** proof that a shared inference process safely isolates users, context, cache and logs.
- Home Assistant, identity, backups, updates, telemetry and administration receive separate restricted security boundaries.
- Every update enters quarantine; require provenance, staged testing, explicit approval, rollout and rollback.
- **AI-assisted and fallible by design:** ChatGPT, Grok, local Hermes-based AI and potentially other agents may contribute and publish over time. Human contributors and all AI systems can make mistakes; preserve reviewable history and analyze errors in both reproduced and newly designed solutions.
- Publish only in **English**, with secret/PII/operational-information review before any public commit.

## Project map
- [Architecture and trust boundaries](docs/ARCHITECTURE.md)
- [Owner-controlled implementation roadmap](docs/ROADMAP.md)
- [EU / Italian legal design baseline](docs/LEGAL-BASELINE.md)
- [Public English-only policy](policies/PUBLICATION-LANGUAGE.md)
- [AI-assisted authorship, fallibility and historical integrity policy](policies/AI-DEVELOPMENT-AND-HISTORY.md)
- [Update quarantine](policies/UPDATE-QUARANTINE.md)
- [Synthetic security test plan](tests/SECURITY-TEST-PLAN.md)

**Security principle:** controls are enforced by deterministic systems outside AI agents. Never treat passing tests as proof of absolute safety. Never expose actual customer material or infrastructure secrets in this public repository.
