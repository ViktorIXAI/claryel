# P2002 — Strata IQ3_S download preparation and failed first launch / GPU-passthrough retry

- **Checkpoint ID:** CLARYEL-P2002-20261106-IQ3S-DOWNLOAD-001
- **Target machine:** P2002 (Ubuntu Server 24.04.5; owner-supplied SSH transcript)
- **Event date:** 2026-11-06
- **Observed event window:** approximately 21:16–21:29 UTC; exact timestamps are available only for some commands and are identified below.
- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Deployment-alias:** P2002-LAB
- **Hermes-runtime-machine-ID:** NOT_APPLICABLE (this entry was prepared by ChatGPT, not a Hermes instance)
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Executed-by:** OWNER_REPORTED_OPERATOR (commands and terminal output supplied by the owner; operator identity and execution were not independently authenticated)
- **Evidence-source:** OWNER_REPORTED
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-06 under the active temporary P2002 technical-log publication authorization
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related checkpoint:** [P2002 Strata network isolation and proxy acceptance](2026-11-06-strata-network-isolation-acceptance.md)
- **Related issue:** [#9 — RX 9070 / Strata bring-up](https://github.com/ViktorIXAI/claryel/issues/9)

> This is a chronological record of owner-supplied terminal output. ChatGPT did not execute commands on P2002. It does not claim a successful model download, successful ROCm compute execution, model load or inference. No passwords, tokens or private keys are recorded.

## 1. Context and target

The preceding network-isolation checkpoint records the P2002 nftables Guard, Docker bridge isolation, Squid proxy, allowed Hugging Face destinations, UFW reload check, service persistence, reboot and bounded positive/negative traffic tests. The owner then moved to downloading the Qwen3.8-Flash-Next IQ3_S model into a persistent Docker volume while retaining the restricted egress path.

Target image:
- `claryel/strata:0.2.11-rocm-gfx120x`
- Strata reported version `0.2.11`, build `0.1.57`, using the embedded Python 3.12.15 runtime at `/opt/strata/terminal/python/bin/python3`.

Target model:
- Repository: `ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-GGUF`
- IQ3_S: two GGUF shards, approximately 83.6 GB total.
- Vision projector: `mmproj-Qwen3.8-Flash-Next-BF16.gguf`, approximately 0.9 GB.
- DFlash2 drafter: approximately 1.2 GB.
- Strata plan total: approximately 85.7 GB.

Persistent storage:
- Docker volume: `claryel-strata-models`
- Container mount: `/models`
- Host volume mount reported by Docker: `/var/lib/docker/volumes/claryel-strata-models/_data`
- Volume creation time reported by Docker: `2026-11-06T21:16:55Z`.
- Available space reported in the container: 304 GB on a 393 GB filesystem (19% used).
- No existing model files were found in the volume before the first launch attempt.

Network policy:
- The download container must use the existing restricted Docker network `claryel-strata-egress`, authorized source address `172.30.250.10`, and the already tested Squid proxy at `172.30.250.1:3128`.
- The approved HTTPS destinations recorded in the network checkpoint are `huggingface.co` and `us.aws.cdn.hf.co`.
- No wildcard proxy allowlist or unrestricted network mode was introduced for the download attempts recorded here.

## 2. Strata plan — read-only, no changes

The owner ran:

```bash
strata launch --family qwen --model iq3_s --data-dir /models --plan --no-tui
```

Strata returned:
- Plan name: `IQ3_S 128K`.
- Model: Qwen3.8-Flash-Next IQ3_S, approximately 84 GB.
- Context: 128K.
- Vision: on, CPU reader.
- Download estimate: approximately 85.7 GB (83.6 GB model, 0.9 GB picture reader, 1.2 GB DFlash2 drafter).
- Tuning: skip.
- Access: local only.
- Port: 8080.

The command ended with `Nothing was changed.` This was planning only; it did not download files or start inference.

## 3. Image and CLI capability inspection

The owner inspected the image using `--network none` and the persistent volume.

Observed:
- `strata` is at `/usr/local/bin/strata`.
- `strata --version` returned `strata 0.2.11 (build 0.1.57; python 3.12.15 at /opt/strata/terminal/python/bin/python3)`.
- The image did not expose `python3` on the shell PATH in the tested invocation.
- The image inspection did not find `curl`, `hf`, `wget`, `aria2c`, `busybox`, `git`, `uv`, `pip` or `pip3` on PATH in the tested command. This is limited to the exact checks shown; it does not prove no internal Strata download implementation exists.
- `strata launch --help` documents model setup, `--model-path`, `--data-dir`, context, vision, drafting and local access options.
- `strata --help` lists separate `install`, `models` and `drafting` command groups.
- `strata models --help` lists `list`, `inspect`, `import` and `move`.
- `strata drafting --help` lists `plan` and `fetch`; no `drafting fetch` was run in this checkpoint.

### 3.1 Installer route rejected by the image

The owner ran `strata install -- --help`. Strata returned:

```text
strata: this folder has no installer (setup.py). Engines, models and updates are installed from a full Strata folder with SETUP.bat / ./setup.sh and UPDATE.bat / ./update.sh; set STRATA_INSTALL_DIR to that folder to run it from here. To update the engine and Control from the signed update feed, run `strata update`
```

The shell used `set -e`, so the command sequence stopped at this error. The subsequent `strata drafting plan --help`, `strata drafting fetch --help` and `strata models import --help` commands in that same shell block did **not** execute. No install or update was performed by this command.

Conclusion: `strata install` was not a usable model-download path from this image directory because the image folder did not contain the full installer/setup.py expected by that command.

## 4. First actual launch attempt — failed before model download

At approximately 21:27:42 UTC, the owner started a detached container named `claryel-strata-iq3s-download` with:
- the restricted egress network and authorized source address;
- proxy environment variables pointing at Squid;
- `HF_HUB_DISABLE_TELEMETRY=1`;
- persistent volume `claryel-strata-models:/models`;
- host log directory `/var/log/claryel` mounted as `/hostlogs`;
- command output appended to `/hostlogs/strata-iq3s-download.log`.

**Important omission:** this first container command did not pass `/dev/kfd` or `/dev/dri` into Docker and did not add the host `render` / `video` group IDs.

The launch command used the planned settings:
- `--family qwen`
- `--model iq3_s`
- `--data-dir /models`
- `--context 128k`
- `--vision cpu`
- `--drafting dflash2`
- `--tune skip`
- `--access local`
- `--port 8080`
- `--yes --no-tui`

Strata output:
- Engine 0.1.57 (HIP) was reported installed.
- It entered “Getting Qwen3.8-Flash-Next IQ3_S ready” and “Checking your computer”.
- It then failed with exit code 5 and this diagnostic:

```text
The installer cannot set up this graphics card yet, and the files Strata would need to write the settings itself are not all there: the model files are not all in /models/models/IQ3_S (every -0000N-of-0000N.gguf shard is needed); the prepared pack /models/packs/iq3_s is missing (this installer cannot prepare it for this graphics card); the tokenizer /models/packs/iq3_s/tokenizer is missing. Use model files that were prepared on this computer already (folder + pack): for an Intel Arc card the steps are in docs/INTEL_ARC.md (the Terminal carries tools/iq_pack.py for the pack). Or install on a computer with an NVIDIA or AMD card.
```

The wrapper then ran `strata launch --stop`; Strata replied `Nothing from strata launch is running.` The log showed 304 GB free before and after, and `find /models` showed no files. Thus this attempt did **not** download the model or start an inference server.

The diagnostic lists missing model shards, prepared pack and tokenizer, and says the installer could not prepare the pack for the detected graphics-card path. The omission of GPU device passthrough in this first Docker run is a concrete configuration defect. It is a plausible contributing factor, but the supplied output does not independently prove that it was the only cause of the installer failure.

## 5. Corrected Docker GPU device passthrough and retry

The owner checked the host devices:
- `/dev/kfd` existed as a character device.
- `/dev/dri` existed and contained `card1`, `card2`, `renderD128` and `renderD129`.
- The host device nodes were owned by root with `render` / `video` groups as shown in the terminal output.

The owner then:
1. Built Docker arguments `--device=/dev/kfd --device=/dev/dri`.
2. Looked up host group IDs for `render` and `video` and added each existing group using `--group-add`.
3. Removed the previous stopped container with the same name.
4. Started a replacement detached container with the same restricted network, authorized source address, proxy variables, telemetry disable flag, image, volume and log mount.
5. Kept the same `strata launch` configuration.

Docker returned container ID:

`f17b3b0b17018d67cc08ee267e1e1a72ab71a2c20c40fa5f25c7e3ceb5311a24`

The owner then inspected its logs. Inside the container, `/dev/kfd`, `/dev/dri/card1`, `card2`, `renderD128` and `renderD129` were visible. The log showed:

```text
=== START STRATA ===
Checking the engine for this computer…
Engine 0.1.57 (hip) is installed.
Getting Qwen3.8-Flash-Next IQ3_S ready (a download may be large; it continues where it stopped if interrupted)…
Getting your model ready
Checking your computer
Getting the helper packages ready
Checking the engine for this PC
```

Docker reported the container `Up` with health `starting` immediately after creation.

**This is the last supplied observation in this checkpoint.** It proves the corrected container started and the GPU device nodes were passed through and visible inside it. It does not yet prove that ROCm enumerated the intended GPU, that model files started downloading, that the pack/tokenizer were prepared, or that inference started.

## 6. Errors and corrections ledger

| Step | Failure / limitation | Correction or disposition | Result |
| --- | --- | --- | --- |
| Image tool inspection | `python3`, curl and HF downloader commands were not on PATH in the tested shell | Inspected Strata's own CLI and embedded Python runtime information | No files changed |
| Installer discovery | `strata install -- --help` reported no `setup.py` in this image folder | Do not use this `install` path from the image directory; inspect/use the supported launch workflow | No install performed |
| First launch container | GPU device nodes and host GPU groups were not passed into Docker | Recreated container with `/dev/kfd`, `/dev/dri`, and available render/video group IDs | Corrected container started |
| First launch setup | Strata reported model shards, prepared pack and tokenizer missing; exit code 5 | No model download was confirmed; retry launched with GPU devices passed through | Awaiting subsequent log evidence |
| Previous assistant diagnosis | Assistant described missing GPU passthrough as the probable cause | Treat this as a hypothesis, not a proven root cause; the corrected run has not yet supplied a final result | Root cause still not fully isolated |

## 7. Current state at last owner-supplied observation

- Persistent volume exists and had 304 GB free before the retry.
- No model files were present before the first attempt.
- First container attempt failed with exit code 5 and was removed before retry.
- Replacement container `claryel-strata-iq3s-download` was started detached; container ID is recorded above.
- GPU device nodes are visible inside the replacement container.
- Strata is still in its computer/helper-package checks in the last supplied log excerpt.
- No IQ3_S shard, mmproj or DFlash2 file has yet been confirmed present or checksum-verified in the supplied output.
- No successful model setup, server start, model load or inference is claimed.
- The host-side log path configured by the command is `/var/log/claryel/strata-iq3s-download.log`.
- The container log can be inspected with `docker logs claryel-strata-iq3s-download`; the volume can be inspected independently by mounting `claryel-strata-models` into a separate `--network none` container.

## 8. Next verification gates

1. Inspect the continuing container log and state; do not remove or restart it while it may be downloading.
2. Check the persistent volume's actual file list and byte sizes. A changing `du` value alone is not proof of a complete shard.
3. Confirm the exact two shard filenames, vision projector, prepared Strata pack and tokenizer required by this Strata build.
4. Confirm official file sizes and repository-provided checksums or pinned hashes; verify each completed file before declaring download success.
5. Inspect the Squid access log during the transfer. If Hugging Face redirects to a destination not in the explicit allowlist, record the exact denied host and assess it individually; do not widen to a wildcard.
6. Confirm whether the current `strata launch --yes --no-tui` command leaves a background serving process running after setup. If it does, stop inference only after the download/setup result is captured; do not report a server stopped unless observed.
7. Run a read-only ROCm/GPU enumeration and a narrowly scoped GPU smoke test before claiming HIP compute acceptance. Device-node visibility inside Docker is not equivalent to a successful ROCm compute test.
8. Update issue #9 only when its stated acceptance criteria have evidence: ROCm sees the intended gfx1201 device, GPU smoke test passes, Strata starts on the GPU, and a model response is verified.

## 9. Explicit non-claims

This checkpoint does **not** claim:
- that the model download began or completed;
- that DFlash2 was fetched;
- that the mmproj file was fetched;
- that the tokenizer or IQ3_S pack was prepared;
- that ROCm successfully enumerated or executed on the RX 9070;
- that inference or an API server started;
- that model checksums passed;
- that the proxy allowlist is sufficient for every Hugging Face/Xet transfer endpoint.

## Contribution provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Hermes-runtime-machine-ID: NOT_APPLICABLE
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: OWNER_REPORTED_OPERATOR (owner-supplied terminal output; not independently authenticated)
- Evidence-source: OWNER_REPORTED
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-06 under the active temporary technical-log publication authorization
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
