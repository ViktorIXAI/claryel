# P2002 / Cloud Hermes — xAI egress preflight and current host/guest snapshot

- **Checkpoint ID:** CLARYEL-P2002-20261107-XAI-EGRESS-PREFLIGHT-001
- **Event time:** 2026-11-07T10:03:40+00:00 (P2002 host); 2026-11-07T10:05:06+00:00 (Cloud Hermes guest)
- **Target host:** P2002
- **Guest VM:** `claryel-cloud-hermes`
- **Guest hostname/user:** `claryel-cloud-vm` / `claryelvm`
- **Guest IPv4:** `172.30.240.12/24`
- **Bridge/network:** `virbr240` / `claryel-cloud-admin`
- **Operation:** READ_ONLY_DIAGNOSTICS
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner on P2002 and inside Cloud Hermes guest; ChatGPT did not access the host or execute host/guest commands
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-6-TRIAGE-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_TRIAGE_AND_REPOSITORY_MAINTENANCE
- **Deployment-alias:** P2002-LAB
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** Owner request for read-only diagnostic collection; no mutation or reboot approval implied
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related issue:** #19
- **Related docs:** [Execution triage](../../docs/EXECUTION-TRIAGE-2026-11-07.md); [Cloud Hermes network/PM/Gateway history](2026-11-07-cloud-hermes-network-pm-repair-gateway.md)

## 1. Executive result

The guest and its Hermes Gateway are operational locally. The Grok failure is explained by the intended network isolation: the guest has only its connected `172.30.240.0/24` route, no default route and no usable DNS scope. Both xAI hostnames resolve on P2002 itself.

The existing Squid proxy does not provide a path for Cloud Hermes. It listens only on `172.30.250.1:3128`, and its ACL permits the Strata-side client `172.30.250.10/32` to CONNECT only to the configured Hugging Face/GitHub download domains. The Guard's `guest_input` chain drops traffic arriving from `virbr240` after its DHCP exception. UFW reports default deny for incoming traffic. Therefore a separate, exact-source, exact-destination proxy path needs coordinated review of Squid, nftables Guard and UFW; no single firewall exception should be assumed sufficient.

No host or guest configuration was changed. No service was restarted. No VM reboot was performed.

## 2. Host state — P2002

Owner-supplied host output at `2026-11-07T10:03:40+00:00`:

- Hostname: `p2002`.
- Kernel: `7.0.0-42-generic`.
- `claryel-guard.service`: active, enabled.
- `docker.service`: active, enabled.
- `libvirtd.socket`: active, enabled.
- `libvirtd.service`: active, enabled.
- `claryel-proxy-firewall.service`: active, enabled.
- `squid.service`: active, enabled.
- Guard verifier output: `PASS: Exact table`, `PASS: Exactly two chains`, `PASS: Chain settings`, `PASS: Exactly 12 ordered rules`, `PASS: No unexpected nftables objects`, `OVERALL: PASS`.

### 2.1 Current Guard rules

The owner-supplied output shows:

```nft
table inet claryel_guard {
    chain guest_input {
        type filter hook input priority filter - 10; policy accept;
        iifname "virbr240" ct state established,related accept
        iifname "virbr240" udp sport 68 udp dport 67 accept
        iifname "virbr240" drop
        iifname "br-strata-eg" ip saddr 172.30.250.10 ip daddr 172.30.250.1 tcp dport 3128 ct state { established, new } accept
        iifname "br-strata-eg" drop
        ip saddr 172.30.250.0/24 drop
    }
    chain guest_forward {
        type filter hook forward priority filter - 10; policy accept;
        iifname "virbr240" drop
        oifname "virbr240" drop
        iifname "br-strata-eg" drop
        oifname "br-strata-eg" drop
        ip saddr 172.30.250.0/24 drop
        ip daddr 172.30.250.0/24 drop
    }
}
```

Guard's exact verifier currently accepts exactly two chains and 12 ordered rules. Any future proxy exception must be made in the canonical Guard configuration and verifier together; do not append an ad-hoc live nftables rule that makes the verifier fail. The current `virbr240` input drop occurs before any xAI proxy exception could exist.

### 2.2 UFW

Owner-supplied output:

- UFW status: active.
- Logging: on (low).
- Default: deny incoming, allow outgoing, deny routed.
- Displayed user rule: SSH TCP/22 on `enp128s31f6` allowed from `192.168.1.118`.

This output does not include the complete UFW before/user/after chains or effective packet path for traffic from `virbr240` to a host-local proxy listener. Inspect those before deciding whether a UFW rule is needed. Do not assume an nftables accept in the Guard base chain overrides a later UFW drop.

### 2.3 Existing Squid

- Listener observed: `172.30.250.1:3128`.
- `squid -k parse` completed and printed the active ACL processing without a parse error.
- Relevant config lines printed by the owner:

```squid
http_port 172.30.250.1:3128
acl strata_client src 172.30.250.10/32
acl allowed_hf dstdomain huggingface.co us.aws.cdn.hf.co github.com codeload.github.com
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access deny blocked_dst
http_access allow strata_client allowed_hf CONNECT
http_access deny all
cache deny all
forwarded_for delete
```

The `blocked_dst` ACL includes loopback, RFC1918, link-local, carrier-grade NAT, documentation, multicast, reserved and IPv6 local/reserved ranges. Existing proxy ACL is intentionally restricted to the Strata download client and the listed download domains. Do not broaden or repurpose this listener for Cloud Hermes.

### 2.4 Host xAI DNS

P2002 successfully resolved:

- `auth.x.ai` → `104.18.18.80`, `104.18.19.80`.
- `api.x.ai` → `104.18.19.80`, `104.18.18.80`.

This proves host-side name resolution at the snapshot time only; it does not prove TLS or application reachability.

### 2.5 Docker evidence gap

The owner ran `docker ps`, `docker inspect` and `docker network inspect` without sufficient permission. Each returned:

```text
permission denied while trying to connect to the Docker API at unix:///var/run/docker.sock
```

Therefore this snapshot confirms the Docker daemon service is active, but does **not** independently confirm the current `claryel-inference` container state, its health, network attachments or API status. Repeat only these read-only queries with `sudo docker` after reviewing the exact command block.

## 3. Libvirt and VM state

Owner-supplied output:

- Network `claryel-cloud-admin`: active, persistent, autostart yes, bridge `virbr240`.
- VM `claryel-cloud-hermes`: running, persistent, autostart enabled, AppArmor enforcing.
- Guest lease: `172.30.240.12/24`.
- Guest MAC: `52:54:00:40:bd:81` (recorded in this authorized P2002 lab log).
- Disk: `/var/lib/libvirt/images/claryel-cloud-hermes.qcow2`.
- No libvirt snapshots were listed.

Autostart is configured but still not proven by a controlled P2002 reboot.

## 4. Backup/recovery evidence and limitation

The backup inventory contains:

| Timestamp (UTC) | File | Size |
|---|---|---:|
| 2026-11-06 15:35:26 | `cloud-hermes-before-netplan.qcow2` | 1,898,774,528 bytes |
| 2026-11-06 15:50:29 | `cloud-hermes-netplan-checkpoint.qcow2` | 1,898,840,064 bytes |
| 2026-11-06 16:30:52 | `cloud-hermes-pre-reboot-20261106.qcow2` | 1,898,906,560 bytes |
| 2026-11-06 16:30:52 | `cloud-hermes-pre-reboot.xml` | 6,615 bytes |

The live VM disk is at `/var/lib/libvirt/images/claryel-cloud-hermes.qcow2`, but this output did not capture its current size, modification time, backing chain or qemu-img health. The listed qcow2 backups predate the 2026-11-07 package-manager repair and Gateway setup recorded in the later guest runtime log. They are not a current recovery point for today's guest state.

No current VM snapshot exists in the `virsh snapshot-list` output. Before reboot, inspect live disk metadata, backup storage capacity and the prior backup verification manifests; then plan a consistent, current offline backup with an explicit stop/start window and owner approval. Do not copy a live qcow2 file as if it were a consistent backup.

## 5. Guest state — Cloud Hermes

Owner-supplied guest output at `2026-11-07T10:05:06+00:00`:

- Hostname: `claryel-cloud-vm`.
- Kernel: `6.8.0-147-generic`.
- IPv4 route: only `172.30.240.0/24 dev enp6s0`, source `172.30.240.12`.
- No default route.
- `systemd-resolved` shows link `enp6s0` with `Current Scopes: none` and `-DefaultRoute`.
- `getent ahostsv4 auth.x.ai` and `api.x.ai` returned no addresses.
- User lingering: `Linger=yes`.
- `hermes-gateway.service`: enabled and active.
- Gateway properties: `MainPID=3380`, `NRestarts=0`, `ExecMainStatus=0`, `ActiveState=active`, `SubState=running`, `UnitFileState=enabled`.
- Hermes CLI: `Hermes Agent v0.23.0`, commit `2bd378ed1b8d`, Python `3.14.9`, OpenAI SDK `2.54.0`, up to date.
- Current model: `grok-4.6`.
- Active provider: `xAI Grok OAuth (SuperGrok / Premium+)`.
- CLI reports primary xAI OAuth credentials present.

No token, credential, environment secret or auth-store content was requested or published. The CLI's credential-present status is not a successful live inference test.

## 6. Current diagnosis

The guest's lack of a default route and DNS scope is consistent with the explicit isolation policy. P2002 itself resolves both required xAI hostnames. The existing Squid listener is on a different bridge and is ACL-restricted to a different source/domain set. The Guard currently drops new traffic from `virbr240` to host-local services, and UFW has a default deny incoming policy.

The immediate Grok failure is therefore explained by missing guest-to-host proxy reachability and missing guest proxy configuration, not by an absent Hermes installation or absent primary OAuth credentials.

## 7. Next read-only preflight — still required

Before drafting a final change set, capture:

1. Complete effective UFW rules and packet path: `sudo ufw status numbered`, `sudo ufw show raw`, and sanitized `sudo iptables-save`/nftables output relevant to INPUT.
2. Exact canonical Guard load/verify files and their systemd units; confirm where the 12-rule invariant is encoded and how a reviewed update is deployed/rolled back.
3. Exact `/etc/squid/squid.conf` with all ACL ordering and any included config files; confirm no hidden ACLs or helper behavior.
4. Docker state with `sudo docker ps`, `sudo docker inspect claryel-inference` (format only non-secret fields), and `sudo docker network inspect claryel-private-ai`.
5. Live qcow2 `qemu-img info`, `stat`, backup storage `df -h`, and relevant checksum/verification manifests. Do not run repair, convert, snapshot or copy commands in this preflight.

Only after those outputs are reviewed should a minimal proxy/Guard/UFW change and rollback be proposed. Any mutation, service reload/restart, guest proxy configuration, backup operation or reboot requires explicit owner approval.

## 8. Provenance and scope

This checkpoint records only the owner-supplied read-only outputs and repository review. ChatGPT did not execute commands on P2002 or inside the guest. No live configuration, firewall, DNS, routing, Squid, Docker, VM, disk or service state was changed. The owner authorized publication of non-secret P2002 lab topology and operational details; no credential material is included.


---

## 9. Follow-up read-only host preflight — owner-supplied terminal transcript

- **Event time:** UNKNOWN — this terminal block did not emit `date -Is`; upload timestamp is not treated as execution time.
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT.
- **Operation:** READ_ONLY_DIAGNOSTICS.
- **Executed-by:** Owner/operator at P2002; identity not independently authenticated.
- **Prepared-by:** ChatGPT GPT-6 (declared).
- **Writer-instance:** CHATGPT-6-TRIAGE-01.
- **Writer-role:** INSTALLATION_TRIAGE_AND_REPOSITORY_MAINTENANCE.
- **Reviewed-by:** SELF_REVIEW_ONLY.
- **Approved-by:** Owner supplied the requested read-only output; no mutation/restart/reboot approval implied.
- **Host or guest changes performed by this checkpoint:** None by ChatGPT; no mutation command was requested or executed.

### 9.1 UFW and iptables packet path

Owner-supplied output shows:

- IPv4 `INPUT` policy is `DROP`.
- The INPUT chain order is:
  1. `LIBVIRT_INP`;
  2. existing explicit Strata proxy exception (`br-strata-eg`, source `172.30.250.10/32`, destination `172.30.250.1/32`, TCP/3128);
  3. UFW before-input and remaining UFW chains.
- `LIBVIRT_INP` currently accepts traffic arriving from `virbr240` only for UDP/TCP port 53 and UDP/TCP port 67. It contains no TCP/3129 exception.
- `ufw-before-input` accepts loopback, established/related, selected ICMP and DHCP replies, then calls `ufw-user-input`.
- `ufw-user-input` contains only the existing SSH rule from `192.168.1.118/32` on `enp128s31f6`, TCP/22.
- UFW status is active; its user-facing rule list likewise shows only that SSH rule.
- IPv4 INPUT policy is DROP. IPv6 INPUT policy is also DROP; no IPv6 proxy listener is proposed.

**Conclusion:** after a precisely scoped Guard accept, a new Cloud Hermes connection to a host-local proxy port would still reach UFW and be dropped unless UFW receives a matching exact allow. The existing libvirt INPUT chain does not itself drop unmatched packets; it simply has no matching TCP/3129 allow, after which the packet continues to UFW. There is no reason in this evidence to modify the libvirt DNS/DHCP rules.

### 9.2 Guard loader and verifier

The current loader at `/usr/local/sbin/claryel-guard-load`:

- checks that `/etc/claryel/firewall.nft` and the verifier exist;
- runs `nft --check --file` against the canonical rules file;
- if the Guard table already exists, verifies the live table but does not replace a present incorrect table;
- if the table is absent, loads the rules file and then verifies.

The verifier is an exact-policy test. It currently expects exactly two chains and 12 ordered rules. Its expected `guest_input` rules include the established/related exception, DHCP exception, then the broad `virbr240` drop. Its source and exact expected rule array were printed in the transcript.

**Important evidence gap:** the transcript displayed the loader and verifier source, but did not display the canonical live `/etc/claryel/firewall.nft` file itself or its current SHA-256. Do not edit the verifier alone or live nftables state alone. The canonical file and verifier must be changed together, with the new exact rule inserted before the `virbr240` drop, and the verifier's exact expected rule count/order updated accordingly.

### 9.3 Squid ACL and listener

The current config excerpt confirms:

- listener: `172.30.250.1:3128`;
- existing source ACL: `172.30.250.10/32`;
- existing allowed domains: `huggingface.co`, `us.aws.cdn.hf.co`, `github.com`, `codeload.github.com`;
- `Safe_ports` and `SSL_ports` are port 443;
- CONNECT is explicitly identified;
- `blocked_dst` is denied before the existing Strata allow;
- all remaining access is denied.

The grep output showed that `blocked_dst` is a multi-line ACL beginning at line 18, but did not capture all its continuation lines. The full file and included configuration order therefore remain to be captured before preparing a final Squid diff.

**Design implication:** Squid `http_access` ACLs are evaluated globally for listeners unless listener/port scope is explicitly applied. A second listener must not accidentally let the existing Strata source/domain rule apply to the new listener, or vice versa. The change plan must scope both listener policies (for example, using a verified `localport` ACL or named-port ACL) and preserve the existing Strata source/domain restrictions. A dedicated second Squid instance is an alternative if a clean single-instance ACL separation cannot be demonstrated.

### 9.4 Docker / Strata state

With `sudo docker`, the owner confirmed:

- `claryel-inference`: running, healthy;
- image: `claryel/strata:0.2.11-rocm-gfx120x`;
- restart policy: `unless-stopped`;
- container address: `172.31.250.2/24`;
- only the internal Docker network `claryel-private-ai` is attached;
- Docker network reports `internal=true`;
- port 8080 is not published to the host.

This closes the earlier Docker permission evidence gap for the fields shown. It does not constitute a full Strata functional/security test; a separate successful inference checkpoint is recorded elsewhere in the repository.

### 9.5 Live VM disk and backup state

- `stat` reported the active Cloud Hermes qcow2 at 5,690,753,024 bytes, with modification time 2026-11-07 10:16:20.059257686 UTC.
- `qemu-img info` refused to open it because it could not obtain the shared write lock while the VM was running.
- This is an expected active-image locking condition, **not evidence that the qcow2 is corrupt**. Do not bypass the lock with an unsafe shared-read option.
- The filesystem containing both the backup and libvirt image paths reports 393 GB total, 151 GB used and 223 GB available (41% used).
- Existing SHA256SUMS manifests cover Guard, Squid/proxy and related configuration artifacts. They do not establish a checksum for a current post-repair Cloud Hermes VM backup.
- The repository's prior 2026-11-06 backup verification is valid only for the then-existing image/checkpoint. Those backups predate the 2026-11-07 Hermes package-manager repair and Gateway setup, so they are not a current recovery point for today's guest state.

**Conclusion:** capacity is sufficient for a current offline copy based on the observed free space, but the live image has not yet been structurally checked in its current state. Use `virsh domblklist` / `virsh domblkinfo` for read-only live metadata. For a byte-consistent offline backup and `qemu-img check`, arrange an owner-approved guest shutdown, then copy and verify the image. Do not stop the VM or create/copy a backup without explicit approval and a defined service window.

### 9.6 Host xAI HTTPS checks

Owner-supplied `curl` results:

- `https://auth.x.ai/.well-known/openid-configuration`: HTTP 200, TLS verification result 0.
- `https://api.x.ai/v1/models`: HTTP 401, TLS verification result 0.
- Both connected successfully to the reported IPv6 address `2606:4700::6812:1250`.

Interpretation:
- The host has working DNS/TLS/network reachability to the xAI OIDC discovery endpoint.
- The API endpoint is also reachable over TLS; HTTP 401 is expected for an unauthenticated request to a protected API endpoint and is not a transport failure.
- This does not yet prove authenticated Grok inference or the guest's ability to use OAuth through a proxy.

### 9.7 Updated minimal design — proposal only

Subject to capturing the complete canonical files and reviewing an exact diff:

1. Add a dedicated Squid listener bound only to host bridge address `172.30.240.1:3129`.
2. Permit only source `172.30.240.12/32`, HTTP CONNECT, destination TCP/443 and exact xAI hostnames `auth.x.ai` and `api.x.ai`; retain the private/reserved destination deny ACL.
3. Explicitly scope the new and existing Squid ACLs to their respective listener ports so the existing Strata/Hugging Face path remains unchanged in effective access.
4. Add one exact Guard input exception for `iifname virbr240`, source `172.30.240.12`, destination `172.30.240.1`, TCP/3129, before the existing `virbr240` drop. Update `/etc/claryel/firewall.nft` and the exact verifier together.
5. Add one exact UFW INPUT allow for source `172.30.240.12`, ingress interface `virbr240`, destination `172.30.240.1`, TCP/3129. No broad libvirt INPUT, FORWARD, NAT or routed allowance is indicated.
6. Keep the guest without a default route and without DNS. The host-side proxy resolves xAI names.
7. Configure only the Hermes Gateway service environment to use the proxy, after confirming the installed HTTP client honors proxy environment variables. Keep credentials out of the service drop-in and logs.
8. Test positive access to the two exact xAI endpoints and negative access to a non-allowlisted public host and private/LAN destinations. Then test OIDC discovery and one real authenticated Grok request.

No item above has been implemented. No firewall, Squid, Docker, Gateway, guest, VM disk, route, DNS or systemd state was changed in this checkpoint.

### 9.8 Remaining read-only capture before final diff

- Print the full canonical `/etc/claryel/firewall.nft`, its `sha256sum`, and compare with the appropriate Guard deployment manifest.
- Print the full Squid config including every continuation line and all `include` directives; capture `squid -k parse` and the listener state.
- Capture UFW configuration flags relevant to IPv6 and user-rule persistence, plus timestamped UFW/Guard unit ordering if needed; do not restart either firewall.
- Capture `virsh domblklist claryel-cloud-hermes --details` and `virsh domblkinfo` for the active disk target (read-only).
- Capture only the Gateway service fragment/drop-in paths and whether proxy environment variables are already configured; never dump OAuth tokens or environment values.
- Confirm the exact Hermes HTTP transport/proxy behavior from the installed pinned source or a safe isolated test.

After this capture, prepare a reviewable patch and rollback sequence. Do not apply it until the owner explicitly approves the mutation and service-reload window.


---

## 10. Follow-up: canonical Guard, full Squid config and live VM block metadata

- **Event time:** UNKNOWN. Squid parser output carries timestamp 2026-11-07 10:31:34; this is the parser's log timestamp, not a separately captured shell start/end time.
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT.
- **Operation:** READ_ONLY_DIAGNOSTICS.
- **Executed-by:** Owner/operator at P2002; identity not independently authenticated.
- **Prepared-by:** ChatGPT GPT-6 (declared).
- **Writer-instance:** CHATGPT-6-TRIAGE-01.
- **Writer-role:** INSTALLATION_TRIAGE_AND_REPOSITORY_MAINTENANCE.
- **Reviewed-by:** SELF_REVIEW_ONLY.
- **Host/guest changes in this checkpoint:** None reported or performed by ChatGPT.

### 10.1 Canonical Guard rules file — captured

The owner displayed the complete current `/etc/claryel/firewall.nft`. Its policy contains:

- `guest_input`, hook INPUT, priority -10, policy ACCEPT:
  1. `virbr240` established/related ACCEPT;
  2. `virbr240` DHCP UDP source 68 / destination 67 ACCEPT;
  3. all remaining `virbr240` input DROP;
  4. exact Strata proxy exception: `br-strata-eg`, source `172.30.250.10/32`, destination `172.30.250.1/32`, TCP/3128, new/established ACCEPT;
  5. remaining `br-strata-eg` input DROP;
  6. source `172.30.250.0/24` DROP.
- `guest_forward`, hook FORWARD, priority -10, policy ACCEPT:
  1. input interface `virbr240` DROP;
  2. output interface `virbr240` DROP;
  3. input interface `br-strata-eg` DROP;
  4. output interface `br-strata-eg` DROP;
  5. source `172.30.250.0/24` DROP;
  6. destination `172.30.250.0/24` DROP.

This is 12 explicit rules across the two chains. The Cloud Hermes input DROP is currently before any exception for a new guest-to-host TCP/3129 connection. The displayed canonical file contains no such exception. The Strata proxy exception remains narrowly scoped.

### 10.2 Guard deployment manifest — hash mismatch requires reconciliation

Owner-supplied live `sha256sum` output:

| Artifact | Live SHA-256 | Deployment manifest SHA-256 | Result |
|---|---|---|---|
| `/etc/claryel/firewall.nft` | `1c1e053348e12cc020fec4626b4663b87cab8ffbddf4f28067a6caa7572c35c2` | `b2549a3861587019c9a8a4d28734ff9dc232f1dc438331831e60f0fd66593d71` | MISMATCH |
| `claryel-guard-verify` | `03426487ac333e29a67f930d165d27a89a5413e0a6f06089fe979d057a3c46df` | `90c9be3cbfd4128cf07fc1f3590517f5a402f4829a8912e0b3426dea5b807bbe` | MISMATCH |
| `claryel-guard-load` | `de002acc3d58ad5637c08a4de2595eab60f77f985391d091b5ad9d88fd0094de` | `de002acc3d58ad5637c08a4de2595eab60f77f985391d091b5ad9d88fd0094de` | MATCH |

The manifest was read from the dated deployment backup directory `guard-deploy-20261106T204013Z`. The loader matches that manifest, while the canonical rules file and verifier do not.

**Interpretation:** this is an unresolved baseline/hash discrepancy. It may reflect a later intentional update, an outdated deployment manifest, or unreviewed file drift. The hashes alone do not establish malicious modification, runtime policy failure, or whether the live verifier's expected rule set matches the canonical file. Do not overwrite or "restore" either file from the old manifest, and do not edit the verifier yet.

**Required reconciliation:** capture the full live verifier and loader source; rerun the verifier; capture `nft list table inet claryel_guard`; compare the live table, canonical file and verifier's exact expected rules; inspect the dated deployment backup contents and timestamps. Only then establish a new reviewed baseline and update its manifest as part of an explicitly approved change.

### 10.3 Full Squid configuration — captured

The complete displayed `/etc/squid/squid.conf` contains:

- One listener: `172.30.250.1:3128`.
- One source ACL: `strata_client = 172.30.250.10/32`.
- HTTPS-only `Safe_ports` and `SSL_ports` (port 443); CONNECT method ACL.
- Allowed domain ACL: `huggingface.co`, `us.aws.cdn.hf.co`, `github.com`, `codeload.github.com`.
- `blocked_dst` ACL covers the displayed IPv4 and IPv6 private, loopback, link-local, carrier-grade NAT, documentation, multicast and reserved ranges.
- Access order: deny unsafe ports; deny CONNECT outside SSL ports; deny blocked destinations; allow only the Strata source + approved domains + CONNECT; deny all.
- No `include` directives were printed by the supplied grep.
- `squid -k parse` printed the configuration processing sequence and no parse error in the supplied output. The shell did not capture its exit code, so this is recorded as **NO_PARSE_ERROR_SHOWN**, not a fully evidenced exit-code PASS.
- `ss` showed a listener on `172.30.250.1:3128`; no listener on 3129 was shown.

**Architecture decision:** issue #19 explicitly requires the existing Strata/Hugging Face proxy ACL and listener to remain unchanged. Therefore the preferred design is a **separate Squid instance with its own configuration, listener, service identity/state paths and logs**, rather than adding a second listener to the existing Squid instance and changing its global ACL evaluation. A separate instance must be validated for non-overlapping PID/cache/log paths and must not share mutable cache/state with the Strata instance. This is a design preference, not an implemented or fully tested configuration.

### 10.4 Active Cloud Hermes VM disk metadata — captured safely

Owner-supplied `virsh domblklist --details` identified the VM's disk target as `vda` and the attached seed medium as `sda`. `virsh domblkinfo` for `vda` returned:

- Capacity: 40.000 GiB.
- Allocation: 5.300 GiB.
- Physical: 5.300 GiB.

This is live block-device metadata only. It is not a backup, checksum, qcow2 consistency check or restore test. No `qemu-img` operation was performed in this checkpoint.

### 10.5 Gateway/proxy inspection — inconclusive because commands ran on the host

The shell prompt identifies the command environment as host `claryel@p2002`. The command `systemctl --user show hermes-gateway.service` returned empty `FragmentPath` and `DropInPaths`; the search path `/home/claryel/.hermes/hermes-agent` did not exist.

These results describe the host user's session/path, not the Cloud Hermes guest's user service. They do **not** establish that the guest Gateway unit is missing or that Hermes lacks proxy support. Guest-side service metadata, proxy environment and installed source remain NOT_CAPTURED.

Next capture must be executed **inside the Cloud Hermes guest**, without printing environment values, OAuth tokens or credentials. Record only proxy variable names as SET/UNSET.

### 10.6 Revised change strategy — proposal only

To honor issue #19's explicit non-goal of changing the existing Strata proxy path:

1. Leave the existing `/etc/squid/squid.conf`, its `172.30.250.1:3128` listener, Strata ACL and service untouched.
2. Evaluate a second independent Squid instance bound only to `172.30.240.1:3129`, with source fixed to `172.30.240.12/32`, CONNECT/443 only, exact domains `auth.x.ai` and `api.x.ai`, private/reserved destination denial, and default deny. Give it separate config, PID, logs and runtime/cache state.
3. After Guard baseline reconciliation, add one exact Guard INPUT exception for source `172.30.240.12`, destination `172.30.240.1`, interface `virbr240`, TCP/3129, before the broad `virbr240` DROP. Update canonical rules and exact verifier together.
4. Add one exact UFW INPUT allow for the same source, destination, ingress interface and TCP port. No FORWARD rule, NAT, default route, broad libvirt allowance or guest DNS.
5. Verify the guest's actual HTTP client/OIDC transport honors the proxy before changing its Gateway service.
6. Test the second proxy in isolation: allowed xAI CONNECT succeeds; another public hostname, private/LAN destinations, wrong source and non-CONNECT methods are denied. Then test OIDC discovery and one authenticated Grok request.
7. Preserve a config rollback copy and owner-approved change window. A current guest disk recovery point is required before any reboot; the dated 2026-11-06 image backup predates the 2026-11-07 Hermes PM/Gateway repair.

### 10.7 Outstanding read-only evidence

- Full current `claryel-guard-verify` and `claryel-guard-load` source.
- Current `claryel-guard-verify` execution result and live `nft list table inet claryel_guard`.
- Contents/metadata of the dated Guard deployment backup and its manifest, to determine why two live hashes differ.
- Guest-side Gateway unit fragment/drop-ins, proxy variable names (not values), Hermes installation path and proxy support in the exact installed version.
- Feasibility details for a second Squid instance: package binary path/version, runtime user, service model, and non-overlapping PID/log/cache directories.

No host or guest configuration was changed. No service was reloaded/restarted, no firewall was mutated, no VM was stopped, and no backup or reboot was performed.


### 10.8 Pinned upstream Hermes source — source-level proxy clue, not guest runtime proof

The public Hermes source at the issue's referenced commit `2bd378ed1b8d6bf2192cb26e0a55ff3091e6bd19` was inspected at `hermes_cli/auth_xai.py`.

- `_xai_oauth_discovery()` calls top-level `httpx.get(XAI_OAUTH_DISCOVERY_URL, ...)`.
- xAI token refresh and device-code flows construct `httpx.Client(...)` without an explicit `trust_env=False`, custom transport or proxy override in the displayed code paths.
- With the standard HTTPX default environment handling, `HTTPS_PROXY=http://172.30.240.1:3129` is expected to be honored for HTTPS OIDC/token requests. This is a source-level expectation only.
- The guest's installed source and HTTPX version have not yet been independently inspected. The inference transport path must also be checked separately; inspecting OAuth code alone does not prove all Gateway inference calls use the same transport or proxy behavior.

Reference: [pinned Hermes xAI OAuth source](https://github.com/NousResearch/hermes-agent/blob/2bd378ed1b8d6bf2192cb26e0a55ff3091e6bd19/hermes_cli/auth_xai.py).

Do not configure the Gateway until guest-side unit metadata, actual installed source/version and runtime proxy environment are captured.


### 10.9 Owner-supplied guest process environment and source-search follow-up

The owner supplied the following read-only output from `claryel-cloud-vm`:

- `systemctl --user show-environment` contained no variables named `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY` or `NO_PROXY`.
- Gateway unit: `LoadState=loaded`, `ActiveState=active`, fragment `/home/claryelvm/.config/systemd/user/hermes-gateway.service`, no drop-ins reported.
- Main Gateway PID: `3380`; `/proc/3380/exe` resolves to `/home/claryelvm/.hermes/tools/python-3.14.9+20261021-linux-x64/bin/python3.14`.
- The Gateway process environment contained no proxy-variable names among the four checked above.
- Process was reported as `hermes`, parent PID 993.
- An attempted `importlib.util.find_spec("hermes_cli.auth_xai")` using the bare Python executable failed with `ModuleNotFoundError: No module named 'hermes_cli'`.
- A filesystem search was run only under `~/.local`, `~/.cache/uv` and `~/.local/pipx`; it returned `SOURCE=NOT_FOUND`.

Interpretation and limitations:

1. There is currently no evidence that the running Gateway receives an HTTP(S) proxy through either the user systemd manager environment or its own process environment. This is a concrete configuration gap for the intended environment-variable-based proxy plan, not yet proof that it is the only reason Grok failed.
2. The Python executable path alone does not identify the installed Hermes module path. Running that interpreter with `-c` also does not reproduce the entry-point script's `sys.path` context. Therefore the `ModuleNotFoundError` is not proof that Hermes is absent or broken.
3. The source search omitted the known Git installation directory `/home/claryelvm/.hermes/hermes-agent` (reported by `hermes --version`) and the service entry point `/home/claryelvm/.hermes/hermes-agent/.hermes/bin/hermes`. The installed source search is incomplete.
4. No Gateway, network, firewall, Squid, VM or boot configuration was changed as part of this diagnostic.

Next read-only checks: search the known Git installation tree and its `.hermes` environment for `hermes_cli/auth_xai.py`; inspect the install commit and entry-point wrapper; inspect the exact unit file while printing only environment variable names (never values); then inspect the installed HTTPX version and OAuth plus inference transport code. Do not mutate the service until those facts are established.
