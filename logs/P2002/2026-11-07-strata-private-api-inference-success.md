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
