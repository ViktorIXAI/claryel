# P2002 — Strata private-network API and verified Qwen inference

**Date:** 2026-11-07 (timestamps below are UTC where present)
**Deployment alias:** P2002-LAB
**Status:** functional inference acceptance PASS; API authentication and complete network-exposure acceptance OPEN.
**Evidence source:** owner-supplied terminal transcripts in the ChatGPT session. The GitHub author did not execute these commands on P2002.

## 1. Runtime readiness

Owner supplied the following Strata server log:

```text
loading the vision encoder ...
loading the model (the first start takes a minute or two) ...
[strata] drafting: DFlash2 alone (--draft dflash2, --draft-conf 0.7)
[strata] starting the engine: reading the model's weights ...
[strata] loading the experts into RAM (about 55 GB) and locking part of them for the GPU.
[strata] experts loaded: 46.84 GiB at 5.82 GiB/s (20 s so far)
[strata] filling the GPU's expert cache (4339 experts, 8.28 GiB of VRAM) ...
[strata] almost ready ...
ready: http://127.0.0.1:8080/v1
(OpenAI: /v1/chat/completions, Anthropic: /v1/messages,
 context 131072 tokens, images on)
from other devices: http://172.31.250.2:8080/
(API: http://172.31.250.2:8080/v1)
WARNING: no API key: anyone who can reach port 8080 can use the model.
```

Observed runtime:
- Container: `claryel-inference`
- Image: `claryel/strata:0.2.11-rocm-gfx120x`
- Internal Docker network: `claryel-private-ai`
- Inference container address in this lab snapshot: `172.31.250.2`
- Model ID: `qwen3.8-flash-next-iq3_s`
- Context reported by Strata: 131,072 tokens
- DFlash2 draft mode enabled, confidence 0.7
- Server announced API readiness and loaded model.
- Container state reported: `running`; health: `healthy`.

A local API check from inside the inference container returned HTTP 200 for `/`, `/v1/models` and `/health`. The model listing reported the model as loaded, text and image input modalities, and a 131,072-token context. Health reported `status=ok`, `loaded=true`, `images=true`, `image_generation=false`, and critically `api_key=false`.

The server log included a DNS-rebinding/Host validation rejection for `Host: claryel-inference:8080` from lab peer `172.31.250.3` (HTTP 403). The same service was reachable by direct IP `172.31.250.2:8080` from a container on the private Docker network (HTTP 200 for `/` and `/v1/models`). This is a Host validation behavior, not caller authentication.

## 2. Initial inference probe — non-accepting result

An initial isolated-client probe returned HTTP 200 and the expected model ID, with 40 completion tokens reported, but the test wrapper printed `ANSWER: None`. That result did not establish a verified answer and was not counted as acceptance. The reason the wrapper observed a null content field was not isolated in that first probe.

## 3. Final isolated-client inference — PASS

The owner then ran a fresh disposable client container with an explicitly assigned address:

```bash
sudo docker run --rm \
  --network claryel-private-ai \
  --ip 172.31.250.10 \
  -e TARGET_IP=172.31.250.2 \
  --entrypoint /opt/strata/terminal/python/bin/python3 \
  claryel/strata:0.2.11-rocm-gfx120x \
  -c '<Python HTTP client: POST /v1/chat/completions, model qwen3.8-flash-next-iq3_s, temperature 0, max_tokens 256, stream false>'
```

The test sent the user prompt:

`Ответь одной короткой фразой: ЛОКАЛЬНЫЙ МОЗГ РАБОТАЕТ.`

Observed response:
- HTTP 200.
- Model: `qwen3.8-flash-next-iq3_s`.
- Assistant content: `ЛОКАЛЬНЫЙ МОЗГ РАБОТАЕТ.`
- Usage: 74 prompt tokens, 101 completion tokens, 175 total tokens.
- Prompt processing: 151.3 tokens/second.
- Generation: 48.6 tokens/second.
- DFlash2: 58 draft tokens, 39 accepted.
- Prompt processing time: 489.1 ms.
- Generation time: 2,078.0 ms.
- Finish reason: `stop`.

The JSON response also contained a `reasoning_content` field. Its content is intentionally not reproduced in this log; the existence of the field is recorded for response-surface and data-exposure review.

**Conclusion:** a separate container attached to `claryel-private-ai` successfully sent a real chat-completion request to the Strata service by direct IP and received the exact expected answer. This proves functional inference across the tested container-to-container path. It does not prove WAN isolation, host-port exposure status, authentication, or resistance to other peers on the Docker network.

## 4. Security findings and unresolved acceptance

1. **No API key is configured.** Strata health explicitly reported `api_key=false`, and startup warned that any peer able to reach port 8080 can use the model.
2. **Host allow-list is not authentication.** The hostname form was rejected by DNS-rebinding protection, but the direct-IP request succeeded. Do not count this 403 as an authorization boundary.
3. **Private-network peer access is demonstrated.** A disposable client on `claryel-private-ai` reached the inference API. The set of other attached peers and their trust level must be inventoried.
4. **Host/WAN publication is UNKNOWN.** The supplied transcript does not include `docker port`, `NetworkSettings.Ports`, host listener inspection or an external reachability test.
5. **Reasoning field is present.** Determine whether `reasoning_content` is intentionally exposed to API clients, whether it can contain sensitive prompt/context material, and whether the public/client API should strip it.
6. **Model access control remains open.** Add a proper authentication boundary (preferably an authenticated gateway or supported API key with protected secret injection) before attaching additional agents or client workloads.
7. **Network isolation remains bounded.** Prior proxy tests and this successful private-network request do not establish every IPv6, DNS/UDP/QUIC, L2, Guard-loss, host/WAN or cross-zone failure case.

Do not expose this unauthenticated endpoint beyond the current tightly controlled lab network. Do not attach additional untrusted agent containers until API authentication and network reachability have been checked.

## 5. Next read-only checks

On P2002, inspect:
- exact published host ports and bindings for `claryel-inference`;
- all network attachments and peer containers on `claryel-private-ai`;
- host listeners on port 8080 and the applicable nftables/Guard policy;
- Strata's supported API-key configuration path and whether it can be enabled without leaking a key into logs or image layers;
- whether the API can suppress or separately control `reasoning_content`.

Then define positive and negative tests:
- authorized caller succeeds;
- caller without a key fails;
- wrong key fails;
- a peer outside the inference zone cannot connect;
- host/WAN cannot reach an unintended published port;
- DNS alias and direct-IP requests are both subject to authentication;
- service restart/recreate retains the same restrictive policy.

No configuration changes, container restarts, firewall changes or host reboots are performed by this log.

## Provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: owner (terminal commands and outputs supplied in conversation); not independently executed by the GitHub author
- Evidence-source: OWNER_SUPPLIED_TERMINAL_OUTPUT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner direction to publish P2002 architecture and non-secret operational details
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_OWNER_SUPPLIED


## 6. Follow-up read-only Strata authentication/source inspection — 2026-11-07

The owner supplied a second terminal transcript. Environment variable values were redacted before sharing; no API key or secret value was published.

### Effective server launch command observed

```text
Path=bash
Args=["-lc","exec strata serve \\
    --profile /data/state/strata/launch/iq3_s-128k.json \\
    --host 0.0.0.0 \\
    --port 8080 \\
    --offline"]
```

This confirms that Strata listens on all interfaces **inside its container**. It does not by itself establish host/WAN reachability; Docker port bindings and host firewall/listener state were not included in this transcript.

### Environment inspection

The owner listed environment variable names while replacing every value with `<redacted>`. The visible list did not include `STRATA_API_KEY` or `STRATA_ALLOWED_HOSTS`. This is consistent with the earlier health result `api_key=false`; no credential value was disclosed.

### Source-code findings from the installed Strata tree

The owner ran a read-only grep of `/opt/strata/terminal`. The output identifies these implementation paths:

- `strata_cli/cli.py`: reads `STRATA_API_KEY` or the configured `api_key` value.
- `strata_cli/launch/settings.py`: documents `api_key` as private preset state, not shown in full.
- `strata_cli/launch/runner.py`: writes the configured key into the server config when present; explicitly writes `api_key = "none"` for the intentional no-key mode.
- `strata_cli/launch/cmd.py`: supports `--api-key-env` and rejects an unset/empty named environment variable.
- `serve/server.py`: documents that when a key is set, `/v1/*` requires Bearer or `x-api-key`; the request handler checks the supplied key with `key_matches`.
- `serve/server.py`: DNS-rebinding/Host validation also supports configured allowed hosts and the `STRATA_ALLOWED_HOSTS` environment variable.

**Assessment:** Strata has a native API-key mechanism; a custom patch to Strata is not the first choice. The currently running service has no effective API key, as independently indicated by its health output. The inspected environment listing contains no `STRATA_API_KEY`. The profile JSON itself was not printed or inspected in this transcript, so its exact stored value is not asserted.

A supported configuration path appears to exist through `STRATA_API_KEY` or the profile's `api_key` setting. Before choosing one, inspect the exact Strata 0.2.11 launch/config lifecycle and the container's mounts so that a key is not placed in a Docker image layer, command line, shell history, public manifest or logs. Prefer a protected secret file or secret injection supported by the actual runtime, with least-privilege read access and a documented rotation procedure. Do not assume Docker environment variables are secret storage.

### What remains unknown

This transcript did **not** include:
- `docker inspect` port bindings / `docker port` output;
- host `ss -ltnp` output;
- `docker network inspect claryel-private-ai` peer inventory;
- effective nftables/Guard rules for port 8080;
- profile JSON's redacted `api_key` status or mount source;
- negative tests for missing/wrong key and positive test with a protected valid key.

Therefore host/WAN exposure and the full reachable-peer set remain UNKNOWN. No configuration, container, firewall, network or host changes were performed.

### Updated next steps

1. Finish read-only Docker port-binding, network-peer, host-listener and Guard/nftables inspection.
2. Inspect the profile's API-key **presence/status only**, without printing its value, and inspect the profile mount source/permissions.
3. Confirm the supported Strata 0.2.11 key injection path and whether container recreation preserves it.
4. Present a concrete implementation and rollback plan before changing the running service.
5. After owner approval, test: no key -> 401/403; wrong key -> 401/403; correct key -> successful completion; both direct-IP and hostname paths remain authenticated.
6. Recheck that no unintended host/WAN port is exposed and that existing Cloud Hermes/Guard paths remain unchanged.



## 7. Docker publication, host listener and network-peer inspection — 2026-11-07

The owner supplied a further read-only terminal transcript.

### Observed Docker configuration

- Container: `claryel-inference`
- Image: `claryel/strata:0.2.11-rocm-gfx120x`
- Exposed container port: `8080/tcp`
- Docker host port bindings: empty (`{}`)
- Docker network publication mapping: `8080/tcp: null`
- `docker port claryel-inference`: no mapping reported.
- Mount `/models`: read-only.
- Mount `/data`: read-write.

The port being declared as exposed in image/container metadata is **not** the same as publishing it on the host. The supplied inspect output shows no host port binding.

### Host listener check

The owner ran:

```bash
ss -ltnp | grep ':8080' || echo "No host listener found on 8080"
```

Observed: `No host listener found on 8080`.

This is evidence that no TCP listener matching port 8080 was visible in that host command output at the time of inspection. It does not rule out direct host-to-container bridge access, other ports, alternate protocols, or a listener that appears under a different namespace/address.

### Private network inspection

Observed:
- Network: `claryel-private-ai`
- Driver: `bridge`
- `Internal=true`
- `IPv6=false`
- Subnet: `172.31.250.0/24`
- Gateway: `172.31.250.1`
- Only listed attached container at inspection time: `claryel-inference`, `172.31.250.2/24`.

The earlier real-inference client was disposable (`docker run --rm`) and had exited before this inventory. Thus the current persistent network inventory contains only the inference container, according to the supplied output.

### Environment variable names

A host-side `docker inspect` listing of environment variable **names only** did not include `STRATA_API_KEY` or `STRATA_ALLOWED_HOSTS`. Values were not printed. This supports the conclusion that the key is not injected through the container environment, but it does not determine whether the mounted profile contains an `api_key` field.

### Incomplete profile inspection

The owner attempted:

```bash
docker exec claryel-inference python3 ...
```

It failed because the image does not provide `python3` on `PATH`. This is a tooling/path issue, not evidence that the profile is absent or unreadable. The previously observed interpreter path is `/opt/strata/terminal/python/bin/python3`; use that exact path for the next read-only profile-status check. Never print the key value.

### Firewall / Guard limitation

The supplied filtered `nft list ruleset` output contained only iptables-nft ownership warnings and no matching lines for the queried port/IP strings. This grep result is **not** a complete firewall-policy review. Generic chain rules, interface-based policy, conntrack rules and Guard behavior were not established. Effective Guard/nftables enforcement remains UNKNOWN.

### Security interpretation

The current evidence establishes:
1. No Docker host-port publication for container TCP 8080 in the inspected configuration.
2. No host TCP listener matching 8080 in the supplied `ss` output.
3. The inference container is attached to an IPv4-only Docker bridge marked `Internal=true`.
4. No other persistent peer was listed on that network at the time of inspection.
5. The container's model mount is read-only; its state/config mount is read-write.

It does **not** establish:
- that the host cannot directly connect to `172.31.250.2:8080`;
- that no other host/WAN route or protocol can reach the container;
- the complete effective firewall/Guard policy;
- whether the mounted Strata profile contains an API key;
- whether every Strata route is authenticated.

No configuration, container, network, firewall, Guard or host changes were performed.

### Next read-only check

Run the profile-status script with the interpreter path already present in the container:

```bash
sudo docker exec claryel-inference /opt/strata/terminal/python/bin/python3 -c '
import json, os, stat
from pathlib import Path
p=Path("/data/state/strata/launch/iq3_s-128k.json")
print("exists:", p.is_file())
if p.is_file():
    s=p.stat()
    print("mode:", oct(stat.S_IMODE(s.st_mode)))
    print("owner_uid:", s.st_uid, "group_gid:", s.st_gid)
    d=json.loads(p.read_text())
    v=d.get("api_key")
    print("api_key_field:", "api_key" in d)
    print("api_key_enabled:", bool(v) and str(v).strip().lower() not in ("none",""))
    print("allowed_hosts_field:", "allowed_hosts" in d)
'
```

This prints only presence/status and file metadata, never the credential. Then inspect the actual Strata route/authentication coverage and prepare a change/rollback proposal. No write action is authorized by this evidence checkpoint.


## 8. Existing Guard evidence versus the new inference subnet

Cross-checking the earlier public Guard records shows that they must not be treated as proof that the new inference subnet is filtered:

- The documented `inet claryel_guard` verifier covers the Cloud Hermes/libvirt interface `virbr240` with its `guest_input` and `guest_forward` chains.
- The documented Strata egress/proxy firewall work refers to the separate Docker network `claryel-strata-egress`, bridge `br-strata-eg`, subnet `172.30.250.0/24`, and authorized proxy client `172.30.250.10`.
- The newly created inference network is `claryel-private-ai`, subnet `172.31.250.0/24`, and the inference endpoint is `172.31.250.2`.

The prior records do not demonstrate a specific Guard/nftables rule for `172.31.250.0/24` or `172.31.250.2`. The latest filtered nftables output also did not show a matching explicit IP/port rule, but that grep is not a complete policy dump.

**Assessment:** Docker's `Internal=true` and single-peer membership are useful isolation controls, but do not substitute for testing host-to-container reachability or proving the effective host firewall path. Treat enforcement of the new inference subnet by CLARYEL Guard as **NOT VERIFIED**, not as PASS.

The next read-only check must:
1. Test direct host-to-container access to `172.31.250.2:8080` without using any proxy.
2. Identify the Docker bridge interface for `claryel-private-ai` and inspect the host route.
3. Inspect the relevant IPv4 INPUT/OUTPUT/FORWARD and Docker chains, including `DOCKER-USER`, plus the full applicable nftables table/chain context.
4. Compare the effective rules with the documented Guard policy and explicitly identify whether the new inference bridge is covered.

No firewall changes are authorized by this finding.


## 9. Direct host reachability and effective firewall/Guard inspection — 2026-11-07

The owner supplied a read-only P2002 terminal transcript. The exact command execution time was not present in the pasted output; no timestamp has been inferred. No secret values were printed.

### Strata profile status

The owner used Strata's bundled interpreter inside `claryel-inference` to inspect only metadata and boolean presence:

- Profile: `/data/state/strata/launch/iq3_s-128k.json`
- File exists: `true`
- Mode: `0600`
- Owner/group: `0:0` (`root:root`)
- `api_key` field: absent
- `api_key` enabled: `false`
- `allowed_hosts` field: absent

The credential value was never printed because no key field exists. This confirms the mounted profile is currently unauthenticated; it does not expose any secret.

### Direct host-to-container access — PASS (reachability confirmed)

The owner ran a direct, proxy-bypassed host request:

```bash
curl --noproxy '*' --connect-timeout 3 --max-time 5 \
  -sS -o /dev/null -w 'HTTP status: %{http_code}\\n' \
  http://172.31.250.2:8080/health
```

Observed result: `HTTP status: 200`.

The owner also supplied the bridge and route output:

- Docker bridge: `br-ffbceca294f9`
- Bridge address: `172.31.250.1/24`
- Container address: `172.31.250.2`
- Host route: `172.31.250.2 dev br-ffbceca294f9 src 172.31.250.1`

**Conclusion:** direct host-to-container API reachability is confirmed. The host is on the bridge's connected network and has a route to the container. This does not establish that another LAN host or the public WAN can reach the container.

### IPv4 iptables policy supplied by the owner

Observed chain policies:

- `INPUT`: `DROP`
- `OUTPUT`: `ACCEPT`
- `FORWARD`: `DROP`

Relevant observations:

- `DOCKER-USER` exists and is empty.
- `DOCKER-FORWARD` contains `-i br-ffbceca294f9 -o br-ffbceca294f9 -j ACCEPT`.
- The existing Strata proxy rules refer to `br-strata-eg` and `172.30.250.0/24`, not the inference bridge.
- The host's `OUTPUT ACCEPT` policy is consistent with the observed host-originated connection to the container. The host-to-container packet is not an inbound connection to a host-local listener, so `INPUT DROP` alone does not block it.
- The same-bridge Docker forwarding rule permits traffic between peers on that bridge; it is not a host egress restriction.

### Full Guard table excerpt supplied by the owner

The owner supplied the complete `inet claryel_guard` table:

- `guest_input` is an input hook at priority `filter - 10`, with policy `accept`.
- Its explicit interface/address rules cover `virbr240`, `br-strata-eg`, and `172.30.250.0/24`.
- `guest_forward` explicitly drops traffic entering/leaving `virbr240` and `br-strata-eg`, and source/destination traffic for `172.30.250.0/24`.
- Neither chain contains an explicit rule for `br-ffbceca294f9`, `172.31.250.0/24`, or `172.31.250.2`.

The owner also listed nftables tables: `ip filter`, `inet claryel_guard`, `ip6 filter`, `ip nat`, `ip6 nat`, and `ip raw`.

**Assessment:** the supplied effective Guard table does not show a rule covering the new inference bridge/subnet. Guard enforcement for `claryel-private-ai` is therefore **NOT VERIFIED / NOT PRESENT IN THE SUPPLIED GUARD TABLE**. This is a precise statement about the displayed table, not a claim that no other firewall mechanism exists.

### Updated security interpretation

1. Docker reports `claryel-private-ai` as `Internal=true` with IPv6 disabled, and previous owner-supplied tests blocked direct container egress to tested public IPv4 endpoints. Those are bounded container-egress observations, not proof of all-protocol isolation.
2. No host port binding for TCP 8080 and no host listener on TCP 8080 were observed in the earlier Docker inspection. The newly confirmed direct host-to-container request is compatible with those facts; Docker bridge routing does not require a published host port.
3. Host-to-container reachability is confirmed (HTTP 200). The host is therefore inside the effective administrative trust boundary unless a separate host OUTPUT/bridge policy is added.
4. The current profile has no API key. Any peer able to reach the API can currently use it; Host/DNS-rebinding validation is not authentication.
5. The current evidence does **not** prove public-WAN or other-LAN reachability. No external-source test was supplied.
6. The supplied Guard table does not explicitly cover the inference bridge. Do not report the new inference subnet as Guard-protected.
7. No firewall, Docker, network, profile, credential, service or host changes were performed.

### Next actions — proposed, not executed

- Preserve the working inference container and its current model/data volumes.
- Prefer Strata's native API-key support over a custom engine patch. Previous owner-supplied source inspection identified Bearer and `x-api-key` enforcement for `/v1/*` when a key is configured.
- Do not place a key in GitHub, an image layer, a command-line argument, shell history, public manifest or ordinary Docker `Config.Env`.
- Prepare a root-owned host secret file with restrictive permissions, mount it read-only into the inference container, and use a minimal protected startup wrapper to read it at runtime and export `STRATA_API_KEY` before executing the existing `strata serve --profile ... --offline` command. This is a proposed pattern only; verify the exact image entrypoint, runtime user, secret-file permissions and Strata 0.2.11 environment precedence before use. The secret file itself must never be printed or committed.
- Before replacing the running container, create and verify a rollback plan that retains the current image, container configuration, model volume, data volume and known-good launch command. Do not delete the current container or volumes until authenticated acceptance passes.
- Test missing key and wrong key rejection, valid Bearer and `x-api-key` success, direct-IP and DNS-name access, and route-by-route behavior. The known source evidence covers `/v1/*`; separately test `/`, `/health`, `/status`, `/metrics`, `/props`, model listing and management routes for information disclosure.
- Re-test host-to-container reachability and confirm the intended trust boundary. If host access is to be blocked, design a narrowly scoped host OUTPUT/bridge policy with a recovery path; do not improvise a live firewall rule.
- Test LAN/WAN reachability from a genuinely external source before making any claim about those zones.

**Change status:** documentation/read-only assessment only. Production API-key enablement and any firewall changes remain unexecuted pending an explicit change window and final preflight.

## Provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: owner (read-only terminal commands and outputs supplied in conversation); GitHub author did not access P2002
- Evidence-source: OWNER_SUPPLIED_TERMINAL_OUTPUT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner direction to continue documenting the P2002 public security architecture
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_OWNER_SUPPLIED


## 10. Exact running-container configuration snapshot — 2026-11-07

The owner supplied the following output from `docker inspect` and `docker image inspect`. This is owner-supplied terminal evidence; the GitHub author did not access the P2002 host.

### Container identity and launch configuration

- Container name: `claryel-inference`
- Full container ID: `30efe186c08874e96413439fae6bf2c46f08255d6725d75df733428ebcfdee5b`
- Image tag: `claryel/strata:0.2.11-rocm-gfx120x`
- Immutable image ID: `sha256:932700126d693662cbd11623555b9c451c7c678da138455b36bb05607f8d061b`
- Repo digest: `claryel/strata@sha256:932700126d693662cbd11623555b9c451c7c678da138455b36bb05607f8d061b`
- Container `Config.User`: empty string (no explicit user value shown in the container configuration). **This does not establish the effective UID until the image's own `Config.User` and the running process identity are inspected.**
- Entrypoint: `["bash"]`
- Command: `bash -lc 'exec strata serve --profile /data/state/strata/launch/iq3_s-128k.json --host 0.0.0.0 --port 8080 --offline'`
- Restart policy: `unless-stopped`
- Network mode: `claryel-private-ai`
- Container address: `172.31.250.2/24`
- Published host port bindings: empty (`{}`)

### Runtime privileges and devices

- `Privileged=false`
- `ReadonlyRootfs=false`
- `CapAdd=null`
- `CapDrop=null`
- `SecurityOpt=null`
- `DeviceRequests=null`
- Additional groups: `993`, `44`
- Explicit device mappings:
  - `/dev/kfd` → `/dev/kfd`, cgroup permissions `rwm`
  - `/dev/dri` → `/dev/dri`, cgroup permissions `rwm`

The empty CapAdd/CapDrop and SecurityOpt values mean no explicit additions/drops/security options were displayed in this inspect output; they do not establish the image's full process privilege requirements or every effective kernel control.

### Health check

- Test: `CMD /opt/strata/healthcheck.sh`
- Interval: 30 seconds
- Timeout: 8 seconds
- Retries: 3
- Start period: 7,200 seconds (2 hours)

Preserve this health check and timing unless a separately tested change is approved.

### Persistent mounts

| Container path | Type / name | Host source | Access |
| --- | --- | --- | --- |
| `/data` | Docker named volume `9fc66e4f52e6c379e22fa03df3e9b685c3f808e5d8ea79a614a4331bb862669b` | `/var/lib/docker/volumes/9fc66e4f52e6c379e22fa03df3e9b685c3f808e5d8ea79a614a4331bb862669b/_data` | Read-write |
| `/models` | Docker named volume `claryel-strata-models` | `/var/lib/docker/volumes/claryel-strata-models/_data` | Read-only |

The two host-side volume root directories were printed as mode `0755`, owner/group `root:root`. This is only the metadata of those two mount-root directories; it does not describe every file below them or the in-container ownership of the Strata profile.

### Incomplete part of the supplied diagnostic

The pasted shell/output becomes truncated or malformed during the mount-permission loop. It shows a subsequent empty source path and a failed `stat` attempt (`cannot statx '': No such file or directory`). The environment-variable-name listing and final container-status command are not reliably present in the supplied output.

Therefore:
- do not infer the environment variable names from this transcript;
- do not infer that the final `docker ps` check completed;
- do not claim that every mount path or file permission was successfully inspected;
- the image's own default `Config.User` and actual in-container process UID/GID remain UNKNOWN.

### Change-planning consequence

The current container's image digest, GPU device mappings, supplemental groups, health check, restart policy, network, port bindings and persistent volume identities are now documented. Any replacement must preserve these settings unless a separately approved and tested change explicitly alters them.

The empty container `Config.User` is not sufficient evidence to choose ownership for a mounted API-key file. Before finalizing a secret-file mount or startup wrapper, inspect the image-level `Config.User`, run a read-only `id` inside the existing container, and verify that the intended runtime identity can read a test file with the proposed numeric owner/group and mode. Do not create or mount a real API key during this preflight.

No container, image, volume, profile, secret, network, firewall or host configuration was changed.

## Provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: owner (terminal commands and output supplied in conversation); GitHub author did not access P2002
- Evidence-source: OWNER_SUPPLIED_TERMINAL_OUTPUT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner direction to continue the P2002 security review
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_OWNER_SUPPLIED


## 11. Image default user and effective process identity — 2026-11-07

The owner supplied a second read-only diagnostic transcript. It closes the earlier runtime-identity gap.

### Image configuration

```text
User=""
Entrypoint=["/usr/bin/tini","--","/opt/strata/entrypoint.sh"]
Cmd=["run"]
```

The image does not specify a non-root default user. The current container configuration also has an empty `Config.User`.

### Effective identity inside the running container

```text
Exec UID: 0
Exec GID: 0
Supplementary groups: [0, 44, 993]
PID 7 python3: Uid 0 0 0 0; Gid 0 0 0 0; Groups 0 44 993
PID 2395 python3: Uid 0 0 0 0; Gid 0 0 0 0; Groups 0 44 993
```

The owner also confirmed:
- Container state: `running`; health: `healthy`.
- Profile `/data/state/strata/launch/iq3_s-128k.json`: mode `0600`, UID:GID `0:0`.
- Host-side `/data` volume root: mode `0755`, UID:GID `0:0`.
- Host-side profile file: mode `0600`, UID:GID `0:0`.
- Host-side `/models` volume root: mode `0755`, UID:GID `0:0`.

### Docker-configured environment variable names

The owner printed variable names only (values were not displayed). The list was:

```text
DEBIAN_FRONTEND
HF_HUB_DISABLE_TELEMETRY
HF_HUB_OFFLINE
HOME
LANG
NVIDIA_DRIVER_CAPABILITIES
NVIDIA_VISIBLE_DEVICES
PATH
PORT
PYTHONNOUSERSITE
PYTHONUNBUFFERED
STRATA_COMPONENTS_DIR
STRATA_IN_DOCKER
STRATA_INSTALL_DIR
TRANSFORMERS_OFFLINE
XDG_CONFIG_HOME
XDG_STATE_HOME
```

`STRATA_API_KEY` is absent from the Docker-configured environment list. This is consistent with the previous health response reporting `api_key=false`. The list does not prove that no process can set an environment variable after container start; it records only the Docker container configuration inspected.

### Consequence for secret-file feasibility

The current image and process run as UID 0/GID 0. A host-owned root:root secret file with mode `0600`, mounted read-only at a dedicated path, is therefore compatible with the currently observed identity from a basic Unix permission perspective. This closes the prior uncertainty about whether a non-root Strata process could read such a file.

This does **not** yet validate the full secret-injection mechanism. Before implementation, inspect the actual `/opt/strata/entrypoint.sh` and verify:
- whether it reads or modifies arguments/environment in a way a wrapper must preserve;
- whether `STRATA_API_KEY` is consumed by the Strata API server in this exact image/version;
- whether the key or environment is logged, copied into child command lines, exposed by diagnostics, or otherwise disclosed;
- whether a read-only secret-file mount and a small wrapper can preserve `tini` as PID 1 and the original `entrypoint.sh run` behavior.

The safest next design candidate is a root-owned mode-0600 secret file mounted read-only, with a minimal root-owned read-only wrapper that reads the key without printing it and then execs the original entrypoint under `tini`. This remains a proposal until the entrypoint and exact runtime behavior are inspected. Do not put the key in Docker Config.Env, command arguments, profile backups, logs, shell history or GitHub.

No secret was generated, mounted or configured. No container, profile, volume, network, firewall or host configuration was changed.

## Provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: owner (read-only commands and output supplied in conversation); GitHub author did not access P2002
- Evidence-source: OWNER_SUPPLIED_TERMINAL_OUTPUT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner direction to continue the P2002 security review
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_OWNER_SUPPLIED


## 12. Installed Strata entrypoint and native API_KEY_FILE support — 2026-11-07

The owner supplied the output of a read-only inspection of the installed image's /opt/strata/entrypoint.sh and source references under /opt/strata/terminal.

### Native entrypoint behavior observed

The installed image entrypoint documents both API_KEY and API_KEY_FILE as supported settings. Relevant source lines:

```sh
APP=/opt/strata
STRATA="$APP/terminal/bin/strata"
KEY_FILE="${STRATA_KEY_FILE:-/data/state/api-key}"
...
key="${API_KEY:-}"
if [ -z "$key" ] && [ -n "${API_KEY_FILE:-}" ]; then
  [ -r "$API_KEY_FILE" ] || die "API_KEY_FILE=$API_KEY_FILE cannot be read."
  key=$(tr -d '\r\n' < "$API_KEY_FILE")
fi
...
[ -n "$key" ] && { set -- "$@" --api-key-env STRATA_LAUNCH_KEY; }
```

The entrypoint therefore has a built-in file-input path. It reads the file into a shell variable and passes the environment-variable name STRATA_LAUNCH_KEY to the strata launch CLI; the secret itself is not placed in the visible CLI argument shown in this code.

The provided excerpt stops at entrypoint source line 240. It does NOT show the remaining code that sets/exports STRATA_LAUNCH_KEY or the final launch/exec behavior. Those details remain to be inspected before relying on this native path.

### API-key source references observed in the installed tree

The owner's grep output showed:
- launch/cmd.py reads the variable named by --api-key-env and assigns its value to the launch settings.
- launch/runner.py writes s.api_key into a run config's api_key field and stores the API key in the private preset mechanism. Therefore, switching the current deployment to the image launcher may persist a copy of the credential in writable /data state; the exact paths, permissions and backup implications must be reviewed before using that route.
- serve/server.py accepts STRATA_API_KEY as the default for its --api-key argument and otherwise reads the profile config's api_key. The API server source uses the environment key or profile value for request authentication.
- serve/server.py documents Bearer and x-api-key authentication for /v1/*.

### Critical distinction: native entrypoint is bypassed by the live container

The live container's inspected configuration is:
- Entrypoint override: ["bash"]
- Cmd: ["-lc","exec strata serve --profile /data/state/strata/launch/iq3_s-128k.json --host 0.0.0.0 --port 8080 --offline"]

Thus the currently running container does NOT invoke the image's default /usr/bin/tini -- /opt/strata/entrypoint.sh path. Its actual launch goes directly to strata serve --profile ....

Consequently, the image's native API_KEY_FILE feature is not active in the current container merely because the image contains it. Do not set API_KEY_FILE and assume the running strata serve command will consume it.

The current direct strata serve path is compatible in principle with the server's native STRATA_API_KEY environment input. A file-reading wrapper that exports STRATA_API_KEY and then execs the exact existing command remains a candidate for the current deployment, but it is not yet validated. Preserve the existing profile, host, port, offline mode, GPU devices, groups, mounts, private network, static address, restart and healthcheck settings.

### Updated implementation decision

Do not replace the current launch path with the image's native API_KEY_FILE launcher without a separate test: that path runs strata launch, may update private presets/run config, and could alter model setup behavior or persist the key in /data.

Next read-only step: inspect the remaining lines of /opt/strata/entrypoint.sh (241 onward), especially all occurrences of STRATA_LAUNCH_KEY, plus the final launch/exec section. Also inspect the exact server argument/key loading and preset/config persistence paths relevant to the current strata serve --profile path.

Then validate a candidate wrapper with a harmless dummy key in a disposable container before creating or installing a production credential. No secret was generated, mounted or configured. No live container, profile, volume, network, firewall or host configuration was changed.

## Provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: owner (read-only commands and output supplied in conversation); GitHub author did not access P2002
- Evidence-source: OWNER_SUPPLIED_TERMINAL_OUTPUT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner direction to continue the P2002 security review
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_OWNER_SUPPLIED


## 13. Complete entrypoint key flow and persistent-secret locations — 2026-11-07

The owner supplied the remaining read-only source excerpt from the installed Strata 0.2.11 image.

### Native entrypoint: exact observed key flow

- Lines 162–167: API_KEY is preferred; if empty and API_KEY_FILE is set, the script checks that the file is readable and reads it into the shell variable key using tr -d CR/LF.
- Line 226: when key is non-empty, the script adds the argument --api-key-env STRATA_LAUNCH_KEY to the strata launch command. The secret value itself is not included in that command-line argument.
- Lines 279–282: the child launch process is started with STRATA_LAUNCH_KEY=key in its environment.
- launch/runner.py, lines 537–550: derive_config() copies the base config, then writes s.api_key into the derived run config's api_key field when present.
- launch/settings.py, lines 134 and 149–152: LaunchSettings has an api_key field documented as stored only in the private preset file; its redacted serialization replaces the value with "set" when with_key=False. This excerpt does not establish the actual file path or filesystem mode of the preset.
- Entrypoint lines 294–296: after launch succeeds, the script runs strata launch --show-key --preset PRESET and redirects its output to KEY_FILE under umask 077. The previously supplied entrypoint excerpt defines the default as /data/state/api-key (overridable with STRATA_KEY_FILE). This is an additional persisted key copy in the writable /data volume. The 077 umask requests restrictive permissions for the newly created file; actual resulting ownership/mode should still be checked on disk.
- Lines 297–299: if no key was supplied, the script warns that Strata generated one and tells the operator how to show it. This is not the case for a supplied API_KEY/API_KEY_FILE value, but the post-launch --show-key file write still occurs.

### Direct strata serve path used by the live container

The server source excerpt shows:
- --api-key defaults to os.environ.get(STRATA_API_KEY, "") (server.py lines 9919–9924).
- The effective raw key is selected as a.api_key or cfg.get(api_key, "") (lines 10035–10045).
- The CLI documentation states Bearer and x-api-key are accepted on /v1/*.

This is consistent with a file-reading wrapper exporting STRATA_API_KEY and then executing the existing direct strata serve --profile command. It avoids invoking the image entrypoint's strata launch workflow and therefore avoids the specific preset/run-config persistence path shown above. The supplied excerpt does not prove that no other server code writes or logs the key; a targeted read-only search of the current serve path and its configuration writes is still required.

### Security interpretation and decision

1. The native API_KEY_FILE input is not a secret-isolation mechanism: the key is read into a shell variable, passed into the launch process environment, saved by launch into a private preset and derived run config, and written to the default /data/state/api-key file after startup.
2. The native path should not be selected until the preset/run-config file paths, ownership/modes, /data backup coverage, and resulting secret copies are explicitly audited against the deployment's storage policy.
3. The direct-serve wrapper is currently the less invasive candidate because it preserves the live command/profile and can feed the server's existing STRATA_API_KEY input. It is not yet accepted: inspect whether the serve process or any helper persists/logs the environment key, test with a harmless dummy key in a disposable container, and verify unauthorized requests are rejected.
4. No production key was generated, copied, mounted or configured. No live container, profile, volume, network, firewall or host setting was changed.

## Provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: owner (read-only commands and output supplied in conversation); GitHub author did not access P2002
- Evidence-source: OWNER_SUPPLIED_TERMINAL_OUTPUT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner direction to continue the P2002 security review
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_OWNER_SUPPLIED


## 14. Read-only persistence and file-permission follow-up — 2026-11-07

The owner supplied the output of a second read-only inspection command. No secret values were printed.

### Entrypoint state paths

- entrypoint.sh lines 31–34 set STATE_DIR to XDG_STATE_HOME or /data/state/strata, KEY_FILE to STRATA_KEY_FILE or /data/state/api-key, PRESET to docker by default, and PORT to 8080 by default.
- The native entrypoint's KEY_FILE default is therefore /data/state/api-key. The current live container overrides the image entrypoint and directly starts strata serve, so the native entrypoint's post-launch key-file write is not part of the current process path. The diagnostic did not inspect whether STRATA_KEY_FILE is configured in the container environment.

### Private run-config writer

The owner-supplied runner.py excerpt (lines 589–615) establishes:
- write_private() creates the parent directory if necessary.
- It writes through a temporary file in the destination directory.
- It applies chmod 0600 to the temporary file (best effort), then atomically replaces the destination.
- launch_dir() resolves to paths.state_base() / "launch" and creates that directory.

This confirms the intended private-file write mechanism for callers of write_private(). It does not, by itself, prove which exact files are written through that helper; the supplied grep did not search for write_private() call sites. The previously recorded live profile path is /data/state/strata/launch/iq3_s-128k.json and its earlier owner-supplied metadata was 0600 root:root. The new find command did NOT re-check that profile: its -name predicates matched basenames containing "preset" or "launch", or exactly "api-key", whereas the profile basename is iq3_s-128k.json. Do not treat that find output as a recursive permission audit.

### Preset persistence remains only partly resolved

The grep found preset-related call sites in launch/cmd.py around lines 307, 984, 989 and 999, and LaunchSettings documents api_key as stored only in the private preset file. However, the supplied output did not show the preset serializer, exact preset path, its file mode, or the call chain that persists the key. These remain open. The grep pattern also omitted write_private(), so it did not identify all private config writer call sites.

### Live server credential handling

- server.py lines 9919–9924 show --api-key defaults to STRATA_API_KEY.
- Lines 10035–10045 select the effective raw key from the CLI/environment value or, if absent, the profile api_key.
- Lines 7754–7762 show the _authorized() helper: with no configured key it permits the request; with a configured key it accepts Bearer or x-api-key and returns HTTP 401 for missing/wrong credentials.
- Lines 7764–7770 show a separate telemetry authorization helper.
- Lines 8079–8084 show /health returns a boolean api_key status, not the key value.
- The targeted grep did not show a direct write of the server API key in the matched server.py lines. This is not proof that no helper, other module, diagnostic, crash path or process inspection can expose it.
- The excerpt did not show where _authorized() is invoked across route handlers. Consequently it does not prove every /v1 route or non-/v1 management route is protected. Route-by-route enforcement remains an acceptance requirement.

### Interpretation and remaining evidence gaps

1. The private run-config writer's 0600/atomic-replace behavior is a positive control, and the previously inspected active profile metadata is 0600 root:root. Root in the container can still read these files.
2. Do not generalize that mode to the separate preset file until its actual write path and resulting file metadata are inspected.
3. The metadata-only find result listed two runtime log files, both 0600 root:root. Because the filename predicate was selective, it is not a complete inventory of /data/state and did not inspect the active profile basename.
4. The direct-serve wrapper remains the preferred candidate, not an accepted change. Next inspect write_private() call sites and preset save implementation; stat the exact active profile and relevant directories; inspect whether /data is included in backups; search direct-serve code and helper modules for secret persistence; and map all route-level authorization checks.
5. No production secret was generated, copied, mounted or configured. No live container, profile, volume, network, firewall or host setting was changed.

## Additional provenance

- Evidence-source: OWNER_SUPPLIED_TERMINAL_OUTPUT (second read-only source/metadata inspection)
- Executed-by: owner (read-only commands and output supplied in conversation); GitHub author did not access P2002
- Reviewed-by: SELF_REVIEW_ONLY
- Attribution-confidence: DECLARED_OWNER_SUPPLIED
