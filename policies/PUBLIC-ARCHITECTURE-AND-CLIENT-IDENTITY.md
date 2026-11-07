# Public architecture disclosure and per-client deployment identity

**Owner decision: 2026-11-07. Standing direction for CLARYEL public technical documentation.**
**Scope:** the P2002 laboratory reference installation and the design of every future CLARYEL Box.
**Status:** documentation policy. It does not certify security or claim that all enforcement mechanisms are deployed.

## 1. Public P2002 laboratory reference

The owner explicitly directs that the P2002 laboratory installation be documented publicly so that independent engineers can inspect and challenge the real architecture and security design.

For the P2002 lab, publish non-secret implementation details, including:
- actual internal IPv4/IPv6 addresses and subnet assignments used by the lab;
- host, VM, container, Docker network, service and internal DNS names;
- service ports, network attachments, routes, proxy paths and firewall/access-control rules;
- versions, image tags/digests, model identifiers, runtime parameters and deployment manifests;
- commands, sanitized terminal output, test cases, failures, corrections, measured results and security findings;
- architecture diagrams, trust boundaries, information flows, threat assumptions and recovery procedures.

The P2002 lab is a public reference environment, not a template whose identifiers are copied verbatim into customer installations. Record whether each fact is owner-supplied, tool-observed, independently verified, proposed or not tested. Do not upgrade owner-supplied terminal evidence into independent verification.

## 2. Material that remains excluded

This public-architecture direction does not authorize publication of:
- passwords, passphrases, API keys/tokens, cookies, session credentials or secret-bearing URLs;
- private SSH/TLS keys, LUKS recovery material, TOTP seeds, vault contents or secret-manager exports;
- real customer documents, private user data, personal communications or unrelated personal identifiers;
- credentials belonging to third parties or any data the owner is not authorized to disclose.

Do not publish a public-WAN address, customer address, MAC/serial/device identity, personal home path or unrelated network identity merely because it appears in a diagnostic dump. Ask whether it is necessary to reproduce the P2002 lab result; otherwise omit it. Secret detection remains a hard block.

## 3. Per-client Box identity and naming

Every installed customer Box must be provisioned as a distinct deployment. Do not copy P2002's concrete names, addresses, identifiers, keys or certificates into a customer deployment.

Generate at install time:
1. A cryptographically random deployment identifier (for example, a UUIDv4 or equivalent high-entropy identifier).
2. Concrete container/runtime names derived from that deployment identifier and the logical service role.
3. A deployment-specific Docker/network namespace name.
4. An available private subnet selected only after checking host, LAN, VPN and routed-network overlap. Do not assume one fixed RFC1918 subnet is safe everywhere.
5. Internal DNS/service records scoped to that deployment.
6. Unique API credentials, certificates and encryption keys, generated locally and never shared between boxes.

Keep logical service roles stable in configuration and APIs (for example, private-agent, external-agent, inference, security-gateway, artifact-quarantine and state-vault). Resolve each role to its generated runtime name/address in a per-install deployment manifest. The manifest containing a customer's actual names, addresses and identity material is customer-local; public examples use synthetic values.

Do not use random names as an access-control mechanism. They reduce collisions and avoid hard-coded cross-install assumptions, but an attacker who knows the complete naming scheme must still be unable to access a service.

## 4. Security must survive full architectural disclosure

Assume an attacker can read this public repository and knows the complete design, default logical roles, software versions and P2002 lab topology.

Security must therefore come from enforceable controls, not obscurity:
- default-deny host and workload network policy;
- explicit zone segmentation and narrowly permitted flows;
- authenticated and authorized service-to-service APIs, with unique per-install identities;
- no unauthenticated inference endpoint and no unintended host/WAN port publication;
- least-privilege container capabilities, mounts, devices and filesystem access;
- no Docker socket or host administration access for inference/model processes;
- independent Security Gateway enforcement for cross-zone operations;
- secret rotation, protected storage, audit logs, tested backup and recovery;
- fail-closed behavior during boot, restart, update, failure and rollback.

A DNS-rebinding/Host-header check is not authentication. A private Docker network is a network boundary, not proof that every peer attached to it is trusted. Every service must authenticate callers where its threat model requires it.

## 5. Publication and evidence handling

- The P2002 lab's non-secret operational identifiers and architecture may be published under this standing owner direction.
- Continue exact-byte secret scanning and manual review. Do not disable Gitleaks or credential checks to permit infrastructure identifiers.
- For customer deployments, publish schemas, generator logic, synthetic examples and test outcomes—not customer-specific deployment manifests or live addresses.
- Preserve the original terminal evidence and clearly identify its source. Publish the exact relevant command and output where the owner has supplied it and no excluded secret is present.
- If a diagnostic dump includes excluded data, publish a reviewed excerpt or a separately prepared reproduction; do not upload the unreviewed dump.
- Keep the historical 2026-11-06 temporary publication authorization as a historical record. This standing owner direction supersedes its expiry for the P2002 non-secret architecture-disclosure scope only.

## 6. Acceptance criteria for the client identity standard

Before any customer deployment is called ready:
- two independently provisioned test Boxes receive different deployment IDs, concrete service/container names and network names;
- subnet selection detects and rejects an overlapping LAN/VPN/host route;
- generated names are unique and all service references resolve through the deployment manifest;
- no P2002 credential, key, certificate, static IP or runtime name is reused;
- unauthenticated API requests fail, authenticated authorized requests succeed, and unauthorized peers fail;
- security tests still pass when the tester is given the full architecture and all generated names/addresses;
- restore/redeployment preserves the deployment identity where intended or performs a documented identity rotation.

## Provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: NOT_APPLICABLE (policy documentation)
- Evidence-source: explicit OWNER_DIRECTION, 2026-11-07
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner direction in current conversation
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_OWNER_DIRECTION
