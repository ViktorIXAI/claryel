# CLARYEL Architecture v0.2 — replaceable, fail-closed local AI platform

**Status:** owner-directed architecture proposal, 2026-11-07. This document describes a target design, not deployed capability or permission to change the live P2002 installation. Architecture v0.1 remains historical context; this proposal generalizes product-specific roles.

## 1. Design objective

CLARYEL is a persistent, privacy-first computing environment in which software implementations can be replaced without changing trust boundaries or losing user-owned state. Hermes, Strata, Qwen, a particular GPU runtime, Home Assistant integration, plugin or model are replaceable products—not architectural foundations.

Stable concepts are: security zones, machine identity, service roles, authenticated APIs, policy, artifact admission, durable state, evidence, backup and recovery.

## 2. Stable service roles; replaceable implementations

Use neutral logical names for long-lived services. Product, model and version belong in a deployment manifest, not in service identity or container name.

| Logical service role | Example neutral identity | Replaceable implementation examples |
| --- | --- | --- |
| Private agent | agent-private-01 | Hermes or another agent framework |
| External/public agent | agent-external-01 | Hermes cloud VM or another agent |
| Local inference | inference-local-01 | Strata, llama.cpp, vLLM or another runtime |
| Model artifact | model-qwen-main (logical alias) | Qwen or another GGUF/model family and quantization |
| Security gateway | security-gateway-01 | Independently maintained policy-enforcing gateway |
| Artifact intake/quarantine | artifact-quarantine-01 | Download broker + isolated scanner/stager |
| Durable agent state | agent-state-vault-01 | Versioned state store/export service |
| Home automation | home-automation-01 | Home Assistant or compatible controller |

These are naming examples, not deployed containers. Existing container names must not be changed until a read-only inventory of references, volumes, networks, service units, monitoring and rollback is complete. Rename as a controlled migration, not cosmetic live mutation.

Each runtime instance also has its own persistent instance ID, role and runtime-machine ID. If it operates on another machine, target-machine ID is a separate field. Never use a product name or shared GitHub account as proof of instance identity.

## 3. Layered architecture and stable contracts

1. **Hardware and host trust base:** UEFI/Secure Boot policy, kernel, IOMMU, storage, GPU driver and ROCm/HIP/CUDA stack. Hardware-specific dependencies are versioned and independently validated.
2. **Compute adapter:** a narrow device/API boundary exposes only required GPU devices and libraries to the inference runtime. No general host access or privileged container.
3. **Inference runtime:** a replaceable implementation behind a versioned inference API. It receives requests and returns model output; it does not receive document-store mounts, shell, MCP/tool execution, Docker socket or general network access.
4. **Model artifact:** weights, tokenizer, vision projector, drafter and runtime pack are independently versioned artifacts. A model manifest binds exact digests to supported runtime, architecture, quantization, context and GPU requirements.
5. **Agent runtime:** replaceable framework implementing a stable agent service contract. Tools and skills are separately authorized capabilities, not ambient runtime privileges.
6. **Security gateway:** independent policy enforcement for all cross-zone requests. Typed operations, authenticated caller identity, authorization, schema validation, size/rate limits, output inspection and audit decisions. Neither agent can change gateway policy.
7. **Durable state plane:** user memory, skills, prompts, workflow definitions, identity metadata and audit history live outside disposable agent containers. Credentials and ephemeral caches are separate.
8. **Artifact admission and update control:** all software and data artifacts enter quarantine before staging or production.
9. **Operations and recovery:** independent backup, restore, health, logging and emergency controls; public Git is not the private evidence vault.

Contracts must be versioned and tested independently from the implementation. Prefer explicit authenticated HTTP/API contracts or a documented typed RPC protocol. Do not depend on undocumented container names, shared writable volumes or direct agent-to-agent sockets.

## 4. Security zones and information flow

Retain the v0.1 separation: host/hypervisor, private zone, external zone, inference zone, Security GTW, device/HA zone, quarantine zone and backup/admin zone.

- Private and external agents never connect directly. All approved cross-zone exchange goes through Security GTW.
- In strict mode, private documents, excerpts, embeddings, identity details and raw private logs are not sent to external/cloud agents.
- Inference is a compute service, not an authority. Model output is untrusted data and cannot authorize tools or policy changes.
- The quarantine zone has no customer data, production secrets, privileged host sockets or route to production workloads.
- Default deny is enforced below the application layer. Agent prompts and container configuration are not security boundaries.

## 5. Artifact supply-chain admission

Every replaceable artifact is untrusted on arrival: OS/kernel/firmware packages, GPU drivers, container images, inference engines, model weights, tokenizers, projectors, drafter packs, agent frameworks, skills, MCP tools, plugins and HA integrations.

### 5.1 Intake → quarantine → staging → promotion

1. **Intake:** a narrow download broker retrieves only explicitly selected sources and versions. Record source URL/registry, publisher, release/tag, source commit, immutable digest, retrieval time and transport result. Never infer registry/source from an image label.
2. **Quarantine:** preserve original bytes and hash in a separate storage/namespace. No production credentials/data, host sockets, privileged devices or broad network. Downloads do not write into production volumes.
3. **Static checks:** signature/attestation verification where available; digest comparison; archive/path traversal checks; SBOM; dependency and license inventory; CVE scan; source/config/static analysis; file-format validation; ClamAV/YARA or equivalent malware scans as supplemental checks.
4. **Dynamic sandbox:** disposable unprivileged environment, synthetic inputs only, constrained network, read-only source artifacts, resource limits and telemetry of filesystem writes, processes, syscalls and network destinations. Do not execute arbitrary installers merely to inspect them.
5. **Staging:** test the exact immutable digest with synthetic data, API compatibility, offline behavior, permissions, network-deny, resource limits, regression, restart and rollback.
6. **Promotion:** human owner explicitly approves the exact version/digest and risk record. Promote atomically; keep the previous known-good version available for rollback. No floating latest tags.
7. **Runtime:** minimal privileges, read-only root filesystem where practical, dropped capabilities, seccomp/AppArmor, explicit mounts/devices, no Docker socket, no ambient network.

A missing signature or SBOM is not an automatic pass or necessarily an automatic rejection: classify the risk, seek independent source/build evidence, pin the digest, inspect source where available and require explicit owner risk acceptance. Never silently equate absence of known malware with trustworthiness.

### 5.2 Model-specific controls

Antivirus scanning is worthwhile as one layer but is not a model-safety proof. GGUF weights are generally data, not native executables; the parser, loader, tokenizer, custom code, helper packages and surrounding scripts are attack surfaces. A clean ClamAV result does not establish benign model behavior, absence of backdoors or resistance to prompt injection.

For every model package:
- pin repository, revision, exact filenames, sizes and SHA-256 hashes from an independently checked source;
- allow only approved formats; reject unexpected executable/archive/script files and path traversal;
- inspect GGUF header, metadata, tensor counts, dimensions and size limits with an updated parser in a sandbox;
- run ClamAV/YARA as supplemental checks and record scanner version/signature database/date;
- load in a disposable unprivileged staging runtime with no secrets, no private data, no tools and no general internet;
- test malformed input, resource exhaustion, prompt injection, data-exfiltration attempts, tool-call-shaped output and unexpected network/filesystem behavior;
- evaluate output policy separately from file integrity.

### 5.3 Strata and container-specific controls

For the current Strata deployment, first establish the actual registry, immutable image digest, upstream source repository/release/commit, build recipe and image publisher. Inspect entrypoint, install/update scripts, dependencies, telemetry, network calls, default capabilities, mounts and model downloader. Generate/obtain SBOM; scan image packages and source; verify signatures/attestations if supplied. If not available, record the gap and use pinned digest plus explicit owner risk acceptance and sandbox testing. Do not replace or restart the currently running P2002 container during analysis.

## 6. Fail-closed service lifecycle — no startup or replacement gaps

Systemd ordering alone is not continuous enforcement. Requires/After can establish start relationships but do not prove that a firewall table remains present or correct after another process changes it.

Required invariants:
- Host/network default-deny policy is established before Docker, libvirt, proxy, workloads or externally reachable interfaces can pass traffic.
- The restrictive base policy exists independently of workload process health. No bootstrap interval may temporarily allow unrestricted egress.
- Ruleset changes use atomic transactions; a failed policy update leaves the last known-good restrictive policy in place.
- Workload namespaces have no default route except explicitly approved paths. Proxy access is a narrow capability, not unrestricted routing.
- Docker/libvirt-created rules cannot supersede or bypass the host security boundary.
- Guard health includes verification of the actual kernel ruleset, not merely a systemd unit reporting active.
- Loss or tampering of Guard must block affected traffic immediately; watchdog/recovery is secondary detection, not the primary barrier.
- Startup, restart, upgrade, rollback, GPU reset, proxy restart and shutdown all preserve the same invariants.
- New and old versions in a blue/green swap are both restricted before either receives traffic; route switching is atomic and rollback preserves policy.

Create a transition matrix for cold boot, interface up/down, nftables reload, UFW operations, Docker/libvirt restart, container create/replace, proxy failure, Guard crash/table deletion, update, rollback, reboot and shutdown. Test each transition using synthetic networks first; live fault injection requires explicit owner approval and verified recovery.

## 7. Replaceable runtime lifecycle

Use immutable versioned deployments and blue/green or canary replacement:
1. Fetch and admit candidate artifact.
2. Instantiate candidate under a staging identity and network policy.
3. Run compatibility, security, health, synthetic task and offline tests.
4. Owner approves exact manifest digest and migration plan.
5. Atomically route the stable logical service identity to the candidate.
6. Monitor health and policy; if failure, route back to retained known-good deployment.
7. Preserve old runtime until acceptance and backup are verified.

Never upgrade by mutating a running container in place. Keep durable state outside the runtime image. Runtime/model/driver compatibility is an explicit manifest matrix, not an assumption.

## 8. Portable agent state and migration

The agent framework is replaceable; user-owned state must not be locked to Hermes internals. Define a versioned vendor-neutral export bundle containing only classified portable data:
- skills source, dependencies, licenses and requested permissions;
- prompts, role/configuration and policy references;
- tool/MCP schemas and connector mappings, never credentials;
- structured memory with provenance, timestamps, confidence, source references, retention and review status;
- workflow/task definitions and user-approved summaries;
- canonical document/source references and embedding metadata; regenerate vector embeddings when the model changes;
- migration manifest, checksums and audit references.

Keep separate:
- portable non-secret configuration;
- user data and memory;
- executable skills/plugins;
- credentials/tokens (never included; reissue separately);
- ephemeral sessions, caches and temporary indexes (normally not migrated).

Imported memory and skills are untrusted. Scan and review for prompt injection, poisoned instructions, stale facts, personal data and excess permissions. Export deterministically, sign/hash, quarantine, map target schema, report unsupported/dropped fields, dry-run on an isolated target with synthetic data, compare behavior and provenance, obtain owner approval, then promote with rollback. Retain the source agent until migration acceptance.

## 9. Container and service naming

Remove product/model names from durable service/container identity over time. Use role + instance + machine alias, e.g. agent-private-01, agent-external-01, inference-local-01, artifact-quarantine-01. Record the actual implementation separately in the deployment manifest: product, version, digest, role, runtime machine ID, network zone and state schema.

Do not perform cosmetic live renames during the current installation. First inventory references in Compose/systemd/libvirt, networks, volumes, health checks, scripts, dashboards and backup/restore. Then create a reversible mapping and controlled migration.

## 10. Unresolved design decisions

- Which registry and upstream source actually supplied the current Strata image, and what evidence does its publisher provide?
- Which signature/attestation formats can each upstream verify? What is the policy when upstream does not sign?
- Which scanners are useful for each artifact class, and what are their false-positive/false-negative limits?
- Which sandbox technology is available on P2002 without granting untrusted code host/GPU privileges?
- Is GPU access required in quarantine, or can file parsing and static checks be separated from GPU runtime testing?
- What exact network invariant prevents a startup race if nftables/Guard is not yet active?
- Which stable API contract should decouple agent and inference vendors?
- What is the canonical agent-memory/export schema, and what is legally/technically exportable?
- Which services need persistent identities, and how are secrets reissued on migration?
- What is the safe migration order for existing Hermes/Strata-named containers?

## 11. Non-goals and deployment guardrails

This document does not authorize renaming, replacing, stopping or restarting current containers; changing P2002 firewall/network/VM; installing scanners or new packages; downloading replacement artifacts; migrating real agent memory; or exposing customer data. Those require separate plans, owner approval, verified backups and recorded acceptance evidence.

## Provenance

Writer-instance: CHATGPT-REVIEW-01
Writer-role: REPOSITORY_AUDITOR
Deployment-alias: P2002-LAB
Prepared-by: ChatGPT GPT-6 (declared)
Executed-by: NOT_APPLICABLE (architecture documentation only)
Evidence-source: OWNER_REQUEST + TOOL_OBSERVED repository inventory
Reviewed-by: SELF_REVIEW_ONLY
Approved-by: owner request to record architecture analysis
Published-by: GitHub connector via shared ViktorIXAI account
Attribution-confidence: DECLARED_UNVERIFIED
