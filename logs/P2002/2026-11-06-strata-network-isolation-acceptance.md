# P2002 — Strata egress isolation, proxy persistence and post-reboot acceptance

- **Checkpoint ID:** CLARYEL-P2002-20261106-NET-ACCEPT-001
- **Target machine:** P2002 (Ubuntu Server 24.04.5)
- **Event window:** 2026-11-06 20:xx–21:02 UTC; exact start times for most diagnostic commands were not present. The reboot journal explicitly records Guard startup at 20:59:58 UTC and the final Squid test entries at 21:02:01–21:02:02 UTC.
- **Publication time:** GitHub commit metadata; not an execution timestamp.
- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Deployment-alias:** P2002-LAB
- **Hermes-runtime-machine-ID:** NOT_APPLICABLE (this contribution was prepared by ChatGPT, not a Hermes instance)
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Executed-by:** OWNER (owner-supplied terminal commands and output; not independently authenticated)
- **Evidence-source:** OWNER_REPORTED
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** explicit owner request to record and publish this P2002 result under the active 24-hour technical-log authorization
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related issues:** #1 publication gate; #2 private evidence ledger; #4 reproducible firewall tests; #7 deferred Guard failure-mode test; #9 RX 9070 / Strata bring-up; #10 Cloud Hermes autostart

> This is a record of owner-supplied terminal evidence. ChatGPT did not execute commands on P2002. The results are not an independent security audit, certification, or proof against every network bypass. The temporary publication authorization permits the non-secret P2002 operational details recorded here; passwords, tokens, private keys and other secret authentication material remain excluded.

## 1. Result summary

**Observed result: the restricted Strata egress path and its startup persistence passed the owner-run positive and negative tests after a real P2002 reboot.**

The tested policy is deliberately narrow:

- Docker network: `claryel-strata-egress`
- Bridge: `br-strata-eg`
- IPv4 subnet: `172.30.250.0/24`
- Gateway / Squid listener: `172.30.250.1:3128`
- One authorized Strata test-container address: `172.30.250.10`
- Unauthorized test-container address: `172.30.250.11`
- Squid permits HTTPS CONNECT only to `huggingface.co` and `us.aws.cdn.hf.co` for the authorized client.
- Direct container egress is denied. Other containers on this bridge cannot use the proxy or reach host SSH.

The Docker network reports `Internal=false`; isolation therefore depends on the host firewall and proxy controls, not on Docker's internal-network flag.

### Final observed checks

| Check | Result |
| --- | --- |
| nftables Guard exact table, two chains and 12 ordered rules | PASS |
| Atomic production policy replacement | PASS |
| Production Guard loader and service validation | PASS |
| Persistent proxy firewall service | PASS |
| Squid enabled and active after reboot | PASS |
| Docker enabled and active after reboot | PASS |
| Guard enabled and active after reboot | PASS |
| Proxy firewall enabled and active after reboot | PASS |
| UFW reload preserves the proxy exception | PASS |
| Authorized client reaches Hugging Face and its approved CDN over HTTPS | PASS |
| Authorized client is denied GitHub by Squid | PASS |
| Authorized client direct IPv4 internet connection is blocked | PASS |
| Unauthorized client is blocked from proxy, host SSH and direct IPv4 internet | PASS |
| Post-reboot boot ordering: Guard → Docker → proxy firewall → Squid | PASS |
| Full IPv6, DNS, UDP, L2 and all failure-mode coverage | NOT_TESTED |

## 2. Initial read-only inspection

The owner supplied the following observations before the final deployment:

- Squid was running but was **disabled** for systemd autostart.
- UFW was active, with default deny for incoming and routed traffic; the displayed user rule allowed SSH from the administrative workstation only.
- IPv4 INPUT policy was DROP.
- The Strata proxy listener was bound only to `172.30.250.1:3128`.
- The Docker network existed with subnet `172.30.250.0/24`, gateway `172.30.250.1`, and `Internal=false`.
- The nftables table `inet claryel_guard` initially protected the Cloud Hermes bridge `virbr240`; it did not yet contain the Strata proxy exception or bridge-drop rules.
- The iptables state already contained a `DOCKER-USER → CLARYEL-STRATA-EGRESS` jump and a Strata egress chain, but the initial recursive search did not find the corresponding persistent script/configuration references.
- The UFW persistent files did not contain the Strata proxy exception. The later dedicated systemd service became the persistence mechanism for that exception.

The owner also supplied the Squid configuration. It listened on the isolated bridge gateway, allowed only source `172.30.250.10/32`, permitted CONNECT to port 443, allowed only `huggingface.co` and `us.aws.cdn.hf.co`, denied private/reserved destinations, denied all other requests, disabled caching and removed the forwarded-for header. `squid -k parse` later passed.

## 3. Candidate policy and verifier errors — preserved chronology

The work deliberately retained failed attempts and corrections.

### 3.1 First candidate-edit attempt aborted

A first Python edit attempted to insert the exact proxy exception into the staged nftables candidate. It failed an assertion because the expected source text did not match the actual candidate. The command had first copied the original candidate files to `.pre-proxy` backups. A subsequent `cmp` confirmed that the candidate and its backups were identical: **the failed attempt made no candidate change**.

A later exact-context replacement inserted the exception only into `guest_input`, before the bridge-wide drop:

```nft
iifname "br-strata-eg" ip saddr 172.30.250.10/32 ip daddr 172.30.250.1/32 tcp dport 3128 ct state { new, established } accept
iifname "br-strata-eg" drop
```

The Cloud Hermes `virbr240` rules and the forwarding chain were retained.

### 3.2 First isolated verifier run failed

The first candidate verifier expected the original 11-rule table and therefore rejected the new 12-rule policy. The kernel JSON was then inspected directly.

Two representation differences were found and corrected in the staged verifier:

1. nftables JSON represents the exact IPv4 host address as a scalar address string, not as a `/32` prefix object.
2. The conntrack state expression is represented as a set containing `established` and `new`, not as the earlier expected list-membership expression.

After those corrections, the isolated network-namespace test loaded the candidate and passed all checks:

- exact table;
- exactly two chains;
- correct chain hooks, priority and policies;
- exactly 12 ordered rules;
- no unexpected nftables objects.

The isolated test printed the resulting 12-rule table, including the proxy exception before the bridge-wide input drop. The forwarding chain continued to drop traffic entering or leaving the Strata bridge and traffic with Strata-subnet source or destination addresses.

### 3.3 Non-root verifier invocation failed, then root verification passed

One invocation of `/usr/local/sbin/claryel-guard-verify` was made without root privileges and failed with:

```text
Operation not permitted (you must be root)
netlink: Error: cache initialization failed: Operation not permitted
```

This was a privilege-context error, not evidence of a bad firewall policy. Running the verifier with `sudo` passed.

### 3.4 Unauthorized-client test expectation was corrected

The first unauthorized-container test attempted an HTTPS CONNECT request and expected Squid to return the literal text `403 Forbidden`. It failed that assertion because the host firewall dropped the TCP connection before Squid could answer. An empty/non-HTTP response was therefore expected from this layer of enforcement.

The test was corrected to check TCP reachability directly. The unauthorized container then failed to connect to all three destinations tested: proxy port 3128, host SSH port 22, and direct internet address `1.1.1.1:443`. All three were reported blocked.

This was a test-harness expectation error, not a demonstrated access-control bypass.

## 4. Production deployment and persistence

The owner supplied a production deployment transcript showing the following sequence:

1. Verify the currently installed Guard policy.
2. Back up the current Guard policy, verifier and loader under `/var/backups/claryel/guard-deploy-20261106T204013Z`.
3. Validate the staged nftables transaction with `nft --check`.
4. Install the new persistent policy and updated verifier.
5. Apply the firewall update as one nftables transaction: delete the old `inet claryel_guard` table and add the complete replacement table in the same transaction.
6. Verify the live table with the new verifier.
7. Run the production Guard loader and verify the systemd Guard service.

The production verifier reported:

```text
PASS: Exact table
PASS: Exactly two chains
PASS: Chain settings
PASS: Exactly 12 ordered rules
PASS: No unexpected nftables objects
OVERALL: PASS
```

The production loader also reported `PASS: CLARYEL guard verified`.

### 4.1 Persistent proxy firewall

The owner then created a dedicated proxy firewall script and systemd service. The service installs the exact INPUT exception for the authorized container and drops other traffic arriving from the Strata bridge. It is ordered after Docker and before Squid.

The owner also added a Squid service drop-in requiring and ordering it after Docker and the proxy firewall, and made Squid part of the Docker/proxy-firewall lifecycle. Squid was enabled for autostart.

The observed effective dependencies included:

- Docker requires and starts after `claryel-guard.service`.
- `claryel-proxy-firewall.service` requires and starts after Docker, and is ordered before Squid.
- Squid requires Docker and the proxy firewall, and starts after both.
- `systemd-analyze verify` returned exit status 0.
- `iptables-save | grep -c -- '--dport 3128'` returned exactly `1`; no duplicate proxy exception was observed.

All four services were later reported enabled and active.

### 4.2 UFW reload test

Before reloading UFW, the owner backed up the current iptables rules to:

`/var/backups/claryel/iptables-before-ufw-test.v4`

The exact proxy exception existed before reload. After `ufw reload`, the same `iptables -C INPUT ...` check still passed. The owner then reran the service and Guard checks; both services were active and the Guard verifier passed.

The exception is not stored as a UFW user rule in `/etc/ufw/user.rules` or `/etc/ufw/before.rules`; it is reinstalled by the dedicated `claryel-proxy-firewall.service`. The UFW reload test demonstrates that this particular reload did not remove the live exception.

## 5. Positive and negative traffic tests before reboot

### Authorized container: `172.30.250.10`

Using the Strata image `claryel/strata:0.2.11-rocm-gfx120x` on `claryel-strata-egress`, the owner supplied these results:

- HTTPS CONNECT to `huggingface.co:443`: TLS connection established.
- HTTPS CONNECT to `us.aws.cdn.hf.co:443`: TLS connection established.
- HTTPS CONNECT to `github.com:443`: denied by Squid with HTTP 403.
- Direct TCP connection to `1.1.1.1:443`: blocked.
- Access to host SSH: blocked.

Squid access-log entries showed `TCP_TUNNEL/200` for the two approved Hugging Face hosts and `TCP_DENIED/403` for GitHub.

The iptables INPUT counters showed packets matching the single authorized proxy exception and packets hitting the bridge DROP rule. The nftables Guard verifier passed.

### Unauthorized container: `172.30.250.11`

After correcting the test described in §3.4, raw TCP connection attempts to the following all failed:

- `172.30.250.1:3128` — proxy;
- `172.30.250.1:22` — host SSH;
- `1.1.1.1:443` — direct internet.

The owner reported `ALL UNAUTHORIZED CONTAINER TESTS PASSED`.

## 6. Real reboot and post-reboot verification

The owner supplied a new SSH session after a real P2002 reboot. The post-reboot status showed:

- Host alias: P2002.
- Kernel reported by the SSH banner: `7.0.0-42-generic`.
- Uptime at the first check: approximately 0 minutes.
- Boot journal: Guard started at 20:59:58 UTC; Docker started at 21:00:09 UTC; proxy firewall started at 21:00:09 UTC; Squid started at 21:00:09 UTC.
- Guard, Docker, proxy firewall and Squid were each reported both `enabled` and `active`.
- Docker network `claryel-strata-egress` existed with subnet `172.30.250.0/24` and gateway `172.30.250.1`.
- Squid listened on `172.30.250.1:3128`.
- The proxy INPUT exception was present.
- The exact 12-rule nftables Guard verifier passed.

The bridge `br-strata-eg` showed `NO-CARRIER` / `DOWN` immediately after boot because no test container was attached at that moment. The Docker network still existed and its gateway address was present. This state is expected for an idle bridge; it is not evidence of a failed proxy service.

### 6.1 Repeated tests after reboot

The owner reran the positive and negative tests after reboot:

**Authorized container `172.30.250.10`:**
- `huggingface.co`: PASS.
- `us.aws.cdn.hf.co`: PASS.
- `github.com`: denied; PASS.
- direct internet to `1.1.1.1:443`: blocked; PASS.

**Unauthorized container `172.30.250.11`:**
- proxy `172.30.250.1:3128`: blocked; PASS.
- host SSH `172.30.250.1:22`: blocked; PASS.
- direct internet `1.1.1.1:443`: blocked; PASS.

The final Squid log excerpt at 21:02:01–21:02:02 UTC again showed two approved `TCP_TUNNEL/200` requests and a GitHub `TCP_DENIED/403` request. The final nftables verifier returned OVERALL PASS.

## 7. Backup audit and unresolved discrepancy

The owner ran a follow-up inventory of the two backup directories.

### Guard deployment backup

`/var/backups/claryel/guard-deploy-20261106T204013Z` contained:

- `firewall.nft`
- `claryel-guard-verify`
- `claryel-guard-load`
- `SHA256SUMS`

All three checksum checks returned OK.

A later live-vs-backup comparison found that the backed-up `firewall.nft` and `claryel-guard-verify` differ from the current production versions, while `claryel-guard-load` is identical. This is consistent with the backup being the pre-proxy deployment snapshot taken before the final 12-rule policy was installed. It must not be described as a byte-identical backup of the current live configuration.

### Proxy persistence backup

`/var/backups/claryel/proxy-persistence-20261106T204217Z` contained:

- `iptables.v4`
- `nft-ruleset.txt`
- `squid.conf`

The backup directory had **no `SHA256SUMS` manifest**. The Squid configuration backup was byte-identical to the current `/etc/squid/squid.conf` in the reported comparison. The missing checksum manifest remains an open documentation/backup-integrity task; this checkpoint does not claim it was repaired.

The current live checksums were recorded by the owner, but the exact checksum values are not reproduced here because the backup comparison above is the relevant finding.

## 8. What this acceptance does and does not establish

### Established by the supplied tests

- The Guard table has the expected 12 ordered rules.
- The authorized test container can reach the Squid proxy and the two configured Hugging Face destinations.
- The same authorized container is denied GitHub and direct IPv4 internet.
- A second container on the same Docker bridge cannot reach the proxy, host SSH or the tested direct IPv4 internet address.
- The proxy firewall exception is recreated by a dedicated systemd unit after Docker starts.
- Squid starts after the proxy firewall.
- The same services and tests passed after a real host reboot.
- The tested UFW reload preserved the live proxy exception.

### Still NOT_TESTED

- IPv6 outbound attempts from both containers, including DNS/AAAA and IPv6 tunnelling.
- DNS leak behaviour and UDP/QUIC paths.
- ICMP and non-TCP protocols.
- Container-to-container L2 reachability and spoofed source/MAC tests.
- All possible SSRF, redirect, DNS-rebinding and proxy CONNECT edge cases.
- Guard table deletion/tampering after Docker has started; this remains deferred under issue #7.
- Failure and recovery behaviour if Docker, the proxy firewall or Squid stops unexpectedly.
- Reboot persistence across a second reboot or after package/service upgrades.
- Cloud Hermes VM autostart and guest DHCP after this reboot; this network checkpoint did not inspect the VM state.
- RX 9070 / ROCm / HIP functionality after this reboot. The boot banner reports kernel 7.0.0-42, but no post-reboot `dmesg`, `rocminfo`, `amd-smi`, `/dev/kfd` or GPU smoke-test output was supplied here. Issue #9 remains open.
- Any actual Qwen model download or inference. The network tests used TLS handshakes and policy checks, not a full 80+ GB model download.

**Acceptance wording:** `P2002 Strata egress isolation and proxy persistence — owner-run post-reboot functional tests PASS; broader network/security assurance remains incomplete.` Do not call this a production security certification.

## 9. Next actions

1. Add a checksum manifest to the proxy persistence backup and verify it.
2. Keep issues #1 and #2 open: the pre-publication gate and private full-fidelity ledger are not deployed.
3. Keep issue #4 open for reproducible synthetic fixtures and the untested protocol/failure cases.
4. Keep issue #7 deferred as instructed.
5. Verify the post-reboot RX 9070 driver and ROCm/HIP path before attempting inference. The current kernel version alone does not prove the earlier firmware/driver issue is resolved.
6. Download the official Strata-supported ISTA-DASLab Qwen3.8-Flash-Next IQ3_S pack only after confirming the model storage location and available space. The upstream source and CLI selection are:
   - `ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-GGUF`
   - `strata launch --family qwen --model iq3_s`
   - The two IQ3_S shards total approximately 83.6 GB; the BF16 vision projector is approximately 0.91 GB. Strata's own plan should be checked before model start.
7. Keep issue #9 open until ROCm sees gfx1201, a GPU smoke test passes, Strata starts on the GPU and the model produces a verified response.

## Contribution provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: P2002-LAB
- Hermes-runtime-machine-ID: NOT_APPLICABLE
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: OWNER (owner-supplied terminal output; no ChatGPT shell execution on P2002)
- Evidence-source: OWNER_REPORTED
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: explicit owner request to record and publish the P2002 result during the temporary technical-log window
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED


## 10. Proposed IQ3_S download command — NOT EXECUTED

The selected source is the official Strata-supported `ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-GGUF` repository. Its IQ3_S folder contains two shards (approximately 54.8 GB and 28.8 GB; 83.6 GB total). The repository root also contains `mmproj-Qwen3.8-Flash-Next-BF16.gguf` (approximately 0.91 GB). The download is intentionally separate from inference: the post-reboot RX 9070 / ROCm state is still unverified under issue #9.

The following command is proposed for the owner to run on P2002. It uses the already tested authorized container address `172.30.250.10`, the Squid proxy, and a persistent Docker volume. It checks for at least 110 GiB free, performs a HEAD-only redirect preflight against the first model shard, and then downloads the two shards plus the vision projector. It does **not** start Strata or access the GPU. If the image has neither the `hf` CLI nor the `huggingface_hub` Python package, it stops without attempting a download. If the HEAD preflight is blocked by a not-yet-allowed Hugging Face/Xet hostname, it stops before the large download; do not widen the Squid allowlist to a wildcard.

```bash
sudo docker volume create claryel-strata-models >/dev/null

sudo docker run --rm \
  --network claryel-strata-egress \
  --ip 172.30.250.10 \
  -e HTTP_PROXY=http://172.30.250.1:3128 \
  -e HTTPS_PROXY=http://172.30.250.1:3128 \
  -e http_proxy=http://172.30.250.1:3128 \
  -e https_proxy=http://172.30.250.1:3128 \
  -e HF_HUB_DISABLE_TELEMETRY=1 \
  -v claryel-strata-models:/models \
  --entrypoint bash \
  claryel/strata:0.2.11-rocm-gfx120x \
  -lc '
set -euo pipefail

REPO=ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-GGUF
DIR=/models/IQ3_S
FILE1=Qwen3.8-Flash-Next-GSQ-RCO-IQ3_S-00001-of-00002.gguf
FILE2=Qwen3.8-Flash-Next-GSQ-RCO-IQ3_S-00002-of-00002.gguf
MMPROJ=mmproj-Qwen3.8-Flash-Next-BF16.gguf

command -v curl >/dev/null
mkdir -p "$DIR"

AVAILABLE_KB=$(df -Pk /models | awk "NR==2 {print \$4}")
if (( AVAILABLE_KB < 110 * 1024 * 1024 )); then
    echo "STOP: less than 110 GiB free in the persistent model volume"
    df -h /models
    exit 1
fi

curl -fsSIL --max-time 60 \
  "https://huggingface.co/$REPO/resolve/main/IQ3_S/$FILE1" >/dev/null
echo "PASS: Hugging Face first-shard redirect preflight"

if command -v hf >/dev/null 2>&1; then
    hf download "$REPO" "IQ3_S/$FILE1" "IQ3_S/$FILE2" --local-dir /models
    hf download "$REPO" "$MMPROJ" --local-dir "$DIR"
elif python3 -c "import huggingface_hub" >/dev/null 2>&1; then
    python3 - <<PY
from huggingface_hub import hf_hub_download
repo = "$REPO"
hf_hub_download(repo_id=repo, filename="IQ3_S/$FILE1", local_dir="/models")
hf_hub_download(repo_id=repo, filename="IQ3_S/$FILE2", local_dir="/models")
hf_hub_download(repo_id=repo, filename="$MMPROJ", local_dir="$DIR")
PY
else
    echo "STOP: neither hf CLI nor huggingface_hub is installed in this image"
    exit 127
fi

test -s "$DIR/$FILE1"
test -s "$DIR/$FILE2"
test -s "$DIR/$MMPROJ"
echo "PASS: all three requested files are present"
du -sh "$DIR"
df -h /models
'
```

Persistent destination: Docker volume `claryel-strata-models`, mounted at `/models`; model folder `/models/IQ3_S`. The same named volume must be mounted by the later Strata launch container. The command has not been run and no model download is claimed by this checkpoint.
