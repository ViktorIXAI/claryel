# P2002 — Strata API authentication change and rollback plan

**Date:** 2026-11-07  
**Target:** P2002 laboratory inference service  
**Status:** PROPOSED ONLY — no secret generated and no live configuration/container/firewall changes performed  
**Related:** [Issue #16](https://github.com/ViktorIXAI/claryel/issues/16), [current inference/security evidence](../logs/P2002/2026-11-07-strata-private-api-inference-success.md)

## 1. Current evidence

Owner-supplied terminal output establishes:

- Strata 0.2.11 serves Qwen3.8-Flash-Next IQ3_S in container claryel-inference.
- A disposable peer on claryel-private-ai completed a real model request.
- The active profile is root-owned, mode 0600, and has neither api_key nor allowed_hosts.
- The service is unauthenticated; Strata health reported api_key=false.
- Prior owner-supplied source inspection identified native Bearer and x-api-key enforcement for /v1/* when a key is configured, with STRATA_API_KEY or profile api_key as configuration sources.
- Direct host-to-container access returned HTTP 200 through the Docker bridge. No host port binding or host TCP listener on 8080 was found in the earlier inspection.
- The displayed Guard table does not include the new inference bridge. Host OUTPUT policy is ACCEPT.

This plan does not treat DNS-rebinding/Host validation as authentication. It does not claim that the host is isolated from its bridge or that LAN/WAN reachability has been tested.

## 2. Security objective

Before any agent or other client is attached, require authentication for model API calls while preserving the current image, model files, data/config volume, private network, and a recoverable known-good launch.

The Strata key is a backend credential for this lab service. It is not a substitute for future per-agent identities, authorization, quotas or a Security Gateway. The intended multi-agent architecture should place an independently authenticated gateway between agents and inference rather than distributing one shared backend key to every agent.

## 3. Read-only preflight required before change

Capture and review the following without printing secret values:

1. Container image ID/digest, user, entrypoint, command, restart policy, health check, capabilities, security options, devices, mounts, network mode and static IP.
2. Exact network attachments and whether the container has any additional network.
3. Host source paths and permissions for the writable data volume and read-only model volume.
4. Current container ID and a tested way to start the existing container again.
5. Confirm API-key absence in the profile and Docker environment by printing field presence and variable names only.
6. Capture a redacted inspect snapshot and checksum of the current profile.

**Stop condition:** if the exact container recreation command, runtime user, mount paths, IP assignment or rollback procedure cannot be reconstructed, do not proceed.

## 4. Proposed secret handling

Prefer Strata's native STRATA_API_KEY support, injected at process start from a protected file, rather than storing the key in an image, Docker command line, shell history, public manifest or ordinary Docker Config.Env.

Proposed pattern, subject to preflight:

- Generate a random high-entropy key locally on P2002. Do not echo, display, copy into chat or commit it.
- Store it in a root-owned host directory mode 0700; the key file should be root-owned mode 0600.
- Bind-mount only that file into the inference container at a fixed path such as /run/secrets/inference-api-key, read-only.
- Use a minimal root-owned startup wrapper that checks the file is present and non-empty, reads it at runtime, exports STRATA_API_KEY, then executes the same Strata serve command with the existing profile, host, port and offline options.
- Verify the actual container runtime UID can read the mounted file. Do not assume it runs as root until Config.User is inspected.
- Keep the secret file outside model and data volumes. Do not place it in public repository content or publication-oriented backups.
- Confirm from installed 0.2.11 source/runtime that the environment value is honored and not logged.
- Do not rely on allowed_hosts for authentication.

This is a proposed pattern only; no secret file or wrapper currently exists as a result of this plan.

## 5. Controlled replacement and rollback

The current container must remain recoverable until acceptance passes.

1. Save the sanitized inspect snapshot, image ID, network configuration, volume names, profile checksum and exact launch command.
2. Verify the existing model and data volumes independently. Do not remove or recreate either volume.
3. Prepare the secret file and wrapper without changing the running container.
4. Prepare a reviewed replacement command that reproduces all current settings and adds only the read-only secret-file mount and wrapper.
5. Schedule a short maintenance window. The current model may need to stop before a replacement can load the same GPU model.
6. Stop and rename (do not delete) the current container as a rollback instance. Create the authenticated replacement with the intended private network/IP and no published ports.
7. If startup or acceptance fails: stop and remove only the replacement container (never remove volumes), restore the original container name and start the original container.
8. Do not delete the rollback container until the owner reviews the evidence and explicitly accepts the new state.

The exact commands are intentionally not finalized until preflight captures the current Docker configuration. Reconstructing a run command from incomplete output risks silently dropping GPU devices, groups, security options, mounts, health checks or restart behavior.

## 6. Acceptance matrix

Run tests from a disposable client attached only to the intended private network. Keep the key out of command arguments and logs.

| Test | Expected result |
| --- | --- |
| GET /v1/models with no key | 401/403 |
| GET /v1/models with incorrect Bearer key | 401/403 |
| GET /v1/models with correct Bearer key | 200 |
| GET /v1/models with correct x-api-key | 200 |
| Real POST /v1/chat/completions with no key | 401/403 |
| Real completion with correct key | 200 and expected response |
| Direct-IP request without key | 401/403 |
| DNS-name request without key | 401/403 or connection denied; never accepted solely because Host is allowed |
| /, /health, /status, /metrics, /props, model listing and management routes | Enumerate individually; document authentication and information disclosure |
| Restart/recreate | Authentication remains enabled; no published port or unintended network appears |
| Container egress | Repeat bounded no-internet tests; do not generalize beyond tested endpoints/protocols |
| Host access | Explicitly record whether host bridge access remains allowed or a separately reviewed host policy blocks it |

A successful /v1/* key test does not automatically prove that non-API web, health or management routes require authentication. If a route remains unauthenticated, minimize its response or place it behind a separate boundary as appropriate.

## 7. Remaining network design

- The host is currently a trusted administrative peer because direct bridge access succeeds. If host-originated access must be blocked except through an audited administration path, design and test a narrow OUTPUT/bridge rule separately.
- The displayed Guard policy covers the Cloud Hermes/libvirt bridge and the separate restricted-proxy bridge, not the inference bridge. Add explicit inference-zone policy only after separate preflight, ordering analysis and rollback design.
- No external LAN/WAN test has been supplied. Test from a genuinely separate source before making any external-reachability claim.
- Do not attach the local multi-agent runtime yet. After API authentication, define per-agent identity and authorization at the Security Gateway; do not distribute the backend key broadly.

## 8. Approval gates

This document records a proposal only. It does not authorize:
- generating or installing a production API key;
- stopping, renaming, recreating or restarting the inference container;
- modifying the profile or Docker environment;
- changing nftables, iptables, UFW, Docker networks or host routes;
- attaching agent containers or exposing the API outside the current lab network.

Obtain explicit owner approval for the exact preflight-reviewed change and maintenance window. Record every command, error, correction, test and rollback result in a new append-only checkpoint.

## Provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: NOT_APPLICABLE (plan only)
- Evidence-source: OWNER_SUPPLIED_TERMINAL_OUTPUT and prior owner-supplied Strata 0.2.11 source inspection
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner direction to continue the security review; this does not authorize the production change
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_OWNER_DIRECTION / OWNER_REPORTED
