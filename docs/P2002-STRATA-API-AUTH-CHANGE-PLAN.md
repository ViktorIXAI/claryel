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
- Direct host-to-container access returned HTTP 200 through bridge br-ffbceca294f9. No host port binding or host TCP listener on 8080 was found in the earlier inspection.
- The displayed Guard table does not include the new inference bridge. Host OUTPUT policy is ACCEPT.
- The current container uses image digest sha256:932700126d693662cbd11623555b9c451c7c678da138455b36bb05607f8d061b, supplemental groups 993 and 44, devices /dev/kfd and /dev/dri, restart policy unless-stopped, and healthcheck /opt/strata/healthcheck.sh (30-second interval, 8-second timeout, 3 retries, 2-hour start period). It has no published ports, is not privileged, has ReadonlyRootfs=false, and has no explicit CapAdd, CapDrop, SecurityOpt or DeviceRequests in the supplied inspect output.
- The writable /data volume is named 9fc66e4f52e6c379e22fa03df3e9b685c3f808e5d8ea79a614a4331bb862669b; the read-only /models volume is claryel-strata-models. Both host-side mount-root directories were shown as 0755 root:root. The transcript became malformed during the mount loop; environment variable names and final docker ps output are NOT VERIFIED.
- Container and image `Config.User` are both empty, with image entrypoint `/usr/bin/tini -- /opt/strata/entrypoint.sh` and command `run`. Owner-supplied in-container checks confirmed the active Python processes run as UID 0/GID 0 with supplementary groups 0, 44, 993. The profile is root:root mode 0600; the host-side `/data` root and profile were respectively 0755 and 0600 root:root; `/models` root was 0755 root:root. This makes a root:root mode-0600 secret file compatible with the observed basic Unix access model, subject to testing the actual mount and wrapper.

This plan does not treat DNS-rebinding/Host validation as authentication. It does not claim that the host is isolated from its bridge or that LAN/WAN reachability has been tested.

## 2. Security objective

Before any agent or other client is attached, require authentication for model API calls while preserving the current image, model files, data/config volume, private network, and a recoverable known-good launch.

The Strata key is a backend credential for this lab service. It is not a substitute for future per-agent identities, authorization, quotas or a Security Gateway. The intended multi-agent architecture should place an independently authenticated gateway between agents and inference rather than distributing one shared backend key to every agent.

## 3. Read-only preflight required before change

Capture and review the following without printing secret values:

1. The container image digest, launch command, restart policy, healthcheck, devices, supplemental groups, network, no-port-publication state and named volume identities are now recorded in the dated log. Capture a fresh sanitized snapshot before any change and compare it with this baseline.
2. Inspect the image-level Config.User, Entrypoint and Cmd, then run read-only id and process identity checks inside the existing container. Container Config.User being empty does not by itself establish the effective runtime UID for this image.
3. Exact mount roots and profile ownership are now confirmed: `/data` root 0755 root:root, profile 0600 root:root, `/models` root 0755 root:root. The permissions of every nested model/data file have not been recursively audited.
4. Docker-configured environment variable names were printed without values; `STRATA_API_KEY` is absent. This is consistent with `/health` reporting `api_key=false`. It does not rule out a process setting an environment variable after startup.
5. Still inspect `/opt/strata/entrypoint.sh` read-only and verify how the exact 0.2.11 image consumes `STRATA_API_KEY`, whether it logs or propagates the value, and whether a wrapper can preserve `tini` as PID 1 and the existing `entrypoint.sh run` behavior.
6. Record current container ID, image digest, all network attachments, and a tested rollback launch procedure. Keep the exact original volumes and static IP. Capture a redacted inspect snapshot and checksum of the current profile.

**Stop condition:** if the exact container recreation command, runtime user, mount paths, IP assignment or rollback procedure cannot be reconstructed, do not proceed.

## 4. Proposed secret handling

Prefer Strata's native STRATA_API_KEY support, injected at process start from a protected file, rather than storing the key in an image, Docker command line, shell history, public manifest or ordinary Docker Config.Env.

Proposed pattern, subject to preflight:

- Generate a random high-entropy key locally on P2002 only after the change is explicitly approved. Do not echo, display, copy into chat or commit it.
- Keep the host-side secret in a root-controlled directory outside the model and data volumes, with directory mode 0700 and secret file owner/group 0:0 mode 0600. The owner-supplied diagnostics confirm the image default and current process run as UID 0/GID 0, so this is compatible with the observed basic Unix permissions.
- Bind-mount only the secret file into the inference container at a fixed path such as `/run/secrets/inference-api-key`, read-only. Verify inside the candidate container that the file is present, owned as expected, mode 0600, and readable by the actual Strata process before loading a real key.
- Use a small root-owned, read-only wrapper. Preserve `/usr/bin/tini` as PID 1 and preserve the current entrypoint semantics (`/opt/strata/entrypoint.sh run`). The wrapper should read the key without printing it, export `STRATA_API_KEY` only in its own process environment, then `exec /opt/strata/entrypoint.sh run`. The exact Docker `--entrypoint`/command arrangement must be tested with a harmless dummy secret first; do not guess it during production replacement.
- Docker-configured environment variable names currently do not include `STRATA_API_KEY`; `/health` reported `api_key=false`. Prior source inspection identified native `STRATA_API_KEY` support, but inspect the actual entrypoint and runtime source for this exact image before relying on it.
- A read-only bind mount prevents writes from the container but does not hide the file from root inside the container. The exported environment value may also be readable through process inspection by sufficiently privileged processes in the same container. Treat the inference container as a trust boundary: compromise of its root process can expose its backend credential. Do not claim the secret is inaccessible to container root.
- Keep the secret file outside model and data volumes. Do not place it in public repository content, ordinary Docker Config.Env, image layers, command arguments, shell history, logs or publication-oriented backups.
- Do not rely on allowed_hosts for authentication.

This remains a conditional proposal. The image and active process identity are now confirmed as root (UID/GID 0), and a root:root mode-0600 mounted file is compatible with those basic permissions. The exact entrypoint behavior, secret consumption/logging behavior, wrapper invocation preserving tini, and process-environment exposure still require read-only source inspection and a harmless dummy-secret test. No production secret file or wrapper currently exists as a result of this plan.

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

The exact commands are intentionally not finalized until preflight captures the current Docker configuration and image default user. Reconstructing a run command from incomplete output risks silently dropping GPU devices, groups, security options, mounts, health checks, restart behavior or the image's default process identity.

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
