# Clean-room, owner-controlled roadmap

No automatic implementation or release authorization follows from GitHub tasks or AI suggestions. Every milestone requires owner approval, threat/risk review, reproducible evidence, rollback and signed-off acceptance. Prior code is untrusted reference only.

1. **M0 — Publish design:** boundaries, threat model, data flow, legal applicability, offline requirements and synthetic acceptance criteria. No secrets.
2. **M1 — Harden offline Ubuntu host:** minimal OS, encryption, access policy, default-deny firewall, KVM and recovery baseline. Demonstrate disconnected operation.
3. **M2 — Isolate Strata/LLM:** inference-only container, model integrity and no direct disk/internet/tools; test with synthetic prompts.
4. **M3 — Private Hermes:** isolated identity, scoped synthetic data vault, no WAN, independent authorization and audit.
5. **M4 — Public Hermes VM:** isolated cloud capable VM with limited egress and no routes to private data, LAN or host administration.
6. **M5 — Security GTW:** sole bridge for both Hermes agents, typed allowlisted requests, no private content export, response buffering, deterministic decisions, no direct channels.
7. **M6 — Home Assistant:** separated device network and local-only operation tests; controlled integrations.
8. **M7 — Quarantined updates:** validation, staging, approvals, rollback and repeatable regression.
9. **M8 — Adversarial audit:** cross-context, cache/logs, tool/prompt injection, leakage, firewall bypass, restart, outage and restore.
10. **M9 — Future configurable hybrid:** disabled by default, explicit individual data transfer policy, legal assessment and user-visible authorization.

Never introduce customer documents before security and legal gates are documented and approved. Record measured throughput, latency, overhead, power and cost instead of unsupported estimates.
