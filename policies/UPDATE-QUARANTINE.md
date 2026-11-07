# All changes and updates are untrusted

Apply to OS/firmware, hypervisor, GPU drivers, container images, Hermes, Strata, model files, Home Assistant, libraries, plugins and GTW.

1. Download via a controlled path into a separate quarantine environment without customer data, production secrets or trusted network reachability.
2. Record immutable version, provenance, signatures/digests, dependencies, licenses and SBOM. Hold uncertain artifacts.
3. Perform static/behavioral checks where feasible and stage offline on synthetic fixtures.
4. Run security regression (network/IPv6/DNS deny, filesystem permissions, API isolation, GTW policy, synthetic exfiltration, offline behavior, rollback and restore).
5. Obtain explicit owner promotion approval after reviewing versioned diff, risks and expected changes. No automatic latest-tag promotions.
6. Deploy with least privilege, preserve rollback image and monitor behavior; revert or quarantine anomalies.
7. No agent can approve its own update, alter its permission policy or bypass the quarantine to access private files or cloud services.

Offline systems require an owner-controlled removable-media or otherwise reviewed update workflow; offline capability must persist after every update.


## Implementation status and required admission profile

This file is a high-level policy, **not proof that a quarantine pipeline exists or is enforced**. The detailed owner-directed design is in [Architecture v0.2, Section 5](../docs/ARCHITECTURE-V0.2-REPLACEABLE-PLATFORM.md#5-artifact-supply-chain-admission); implementation/analysis is tracked by [issue #11](https://github.com/ViktorIXAI/claryel/issues/11).

The policy applies to every artifact, not only application binaries: container images, source releases, OS/kernel/firmware/GPU packages, inference runtimes, model weights, tokenizers, vision projectors, drafter packs, agent frameworks, skills, MCP tools, plugins and HA integrations. Require source and immutable digest, publisher/signature evidence where available, SBOM/dependency/license inventory, vulnerability review, file-format validation, sandbox testing, explicit owner promotion and rollback. Missing signatures or scanner coverage must be disclosed as a risk, not silently marked PASS.

ClamAV/YARA or equivalent file scanning is a supplemental signal. It does not establish model weights are benign, does not test model behavior, and does not replace sandboxed parsing, provenance/hash verification, prompt-injection/exfiltration evaluation or runtime least privilege. Executable skills and installers require code/dependency review and must never be run with production credentials in quarantine.

No current P2002 component is to be replaced, stopped or restarted as part of the policy clarification. The current live Strata installation remains outside this documentation-only change.