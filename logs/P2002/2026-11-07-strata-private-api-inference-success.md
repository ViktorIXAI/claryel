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
