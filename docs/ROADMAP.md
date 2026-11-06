# Clean-room, owner-controlled roadmap

No automatic implementation or release authorization follows from GitHub tasks or AI suggestions. Every milestone requires owner approval, threat/risk review, reproducible evidence, rollback and signed-off acceptance. Prior code is untrusted reference only.

1. **M0 — Publish design:** boundaries, threat model, data flow, legal applicability, offline requirements and synthetic acceptance criteria. No secrets.
2. **M1 — Establish P2002 test host:** install Ubuntu Server 24.04 with a 350 GB **unencrypted root for this test phase only**, protect SSH and network access, set default-deny boundaries, install KVM and document initial recovery assumptions. Use synthetic data only. Encrypted private data and secret volumes remain separately designed. Demonstrate disconnected operation.
3. **M2 — Isolate Strata/LLM:** inference-only container, model integrity and no direct disk/internet/tools; test with synthetic prompts.
4. **M3 — Private Hermes:** isolated identity, scoped synthetic data vault, no WAN, independent authorization and audit.
5. **M4 — Public Hermes VM:** isolated cloud capable VM with limited egress and no routes to private data, LAN or host administration.
6. **M5 — Security GTW:** sole bridge for both Hermes agents, typed allowlisted requests, no private content export, response buffering, deterministic decisions, no direct channels.
7. **M6 — Home Assistant:** separated device network and local-only operation tests; controlled integrations.
8. **M7 — Quarantined updates:** validation, staging, approvals, rollback and repeatable regression.
9. **M8 — Adversarial audit:** cross-context, cache/logs, tool/prompt injection, leakage, firewall bypass, restart and outage. Defer the integrated destructive recovery exercise to M8-R.
10. **M8-R — Integrated backup, destructive restore and root encryption exercise (future, separately approved):** design and verify independent backups and a trial restore first; after explicit owner wipe approval, erase the P2002 test system, perform a fresh LUKS2-encrypted Ubuntu installation, and restore the system, models, services, data and credentials from backup. **No in-place encryption.** Validate reboot, swap, isolation, offline operation and recovery. Not part of initial deployment and not authorized for execution yet.
11. **M9 — Future configurable hybrid:** disabled by default, explicit individual data transfer policy, legal assessment and user-visible authorization.

Never introduce customer documents before security and legal gates are documented and approved. Record measured throughput, latency, overhead, power and cost instead of unsupported estimates.

A recovery test must not begin with disk erasure. Verify independent backup media, restore drills, full scope, recovery keys, target-disk inventory and specific owner approval before any irreversible operation. Do not assume that a successful test proves complete legal or security compliance.
