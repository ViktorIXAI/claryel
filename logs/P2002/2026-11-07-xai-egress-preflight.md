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
