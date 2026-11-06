# P2002 — observed UFW, Docker, Guard and libvirt activation timestamps

- **Checkpoint ID:** CLARYEL-P2002-20261106-UFW-003
- **Event time UTC:** UNKNOWN (terminal paste has no wall-clock timestamp).
- **Target-machine-ID:** P2002
- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Hermes-runtime-machine-ID:** NOT_APPLICABLE (ChatGPT-authored)
- **Deployment-alias:** NOT_APPLICABLE
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Executed-by:** UNKNOWN (owner-supplied P2002 shell transcript; assistant had no remote shell)
- **Evidence-source:** OWNER_REPORTED
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** owner's 2026-11-06 temporary technical-log publication authorization.
- **Published-by:** GitHub connector with shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Host change status:** NONE (diagnostics only)
- **Exit codes:** UNKNOWN; no per-command exit status printed.
- **Related checkpoint:** CLARYEL-P2002-20261106-UFW-002

## Read-only commands supplied by owner

```bash
for unit in ufw.service claryel-guard.service docker.service libvirtd.service; do
  systemctl show --no-pager "$unit" \
    -p ActiveEnterTimestampMonotonic \
    -p ExecMainStartTimestampMonotonic \
    -p Result
done
sudo journalctl -b --no-pager -o short-monotonic \
  -u ufw.service -u claryel-guard.service | tail -60
sudo grep -E '^(ENABLED|IPV6|MANAGE_BUILTINS)=' \
  /etc/ufw/ufw.conf /etc/default/ufw
sudo /usr/local/sbin/claryel-guard-verify
systemctl is-active ssh
```

## Owner-provided timestamp results (microseconds since current boot)

```text
--- ufw.service ---
Result=success
ExecMainStartTimestampMonotonic=6629094
ActiveEnterTimestampMonotonic=9370225
--- claryel-guard.service ---
Result=success
ExecMainStartTimestampMonotonic=8739269388
ActiveEnterTimestampMonotonic=8739310182
--- docker.service ---
Result=success
ExecMainStartTimestampMonotonic=724306275
ActiveEnterTimestampMonotonic=724653841
--- libvirtd.service ---
Result=success
ExecMainStartTimestampMonotonic=14011181417
ActiveEnterTimestampMonotonic=14011190520
```

| Service | Observed start after boot | Observed active state after boot |
| --- | ---: | ---: |
| UFW | 6.629094 s | 9.370225 s |
| Docker | 724.306275 s | 724.653841 s |
| CLARYEL Guard | 8739.269388 s | 8739.310182 s |
| libvirtd | 14011.181417 s | 14011.190520 s |

These are current-boot monotonic timestamps, **not** wall-clock installation times and **not** an end-to-end reboot test of the currently configured dependency graph.

## Owner-provided boot journal excerpt

```text
[    6.629116] p2002 systemd[1]: Starting ufw.service - Uncomplicated firewall...
[    9.370236] p2002 systemd[1]: Finished ufw.service - Uncomplicated firewall.
[ 8739.269603] p2002 systemd[1]: Starting claryel-guard.service - CLARYEL isolated VM firewall guard...
[ 8739.288963] p2002 claryel-guard-load[10327]: Existing guard detected; verifying policy
[ 8739.308061] p2002 claryel-guard-load[10330]: PASS: Exact chains
[ 8739.308061] p2002 claryel-guard-load[10330]: PASS: Chain settings
[ 8739.308061] p2002 claryel-guard-load[10330]: PASS: Ordered rules
[ 8739.308061] p2002 claryel-guard-load[10330]: OVERALL: PASS
[ 8739.309875] p2002 claryel-guard-load[10327]: PASS: CLARYEL guard verified
[ 8739.310269] p2002 systemd[1]: Finished claryel-guard.service - CLARYEL isolated VM firewall guard.
```

The loader reported **Existing guard detected** when its service started. This means a guard table existed at that point. Its initial creation time remains UNKNOWN; these logs **do not** demonstrate that the nftables policy was absent while Docker was running.

## UFW configuration settings reported

```text
/etc/ufw/ufw.conf:ENABLED=yes
/etc/default/ufw:IPV6=yes
/etc/default/ufw:MANAGE_BUILTINS=no
```

Interpretation: UFW was enabled, configured to support IPv6, and not configured to manage built-in chains through its `MANAGE_BUILTINS` option. Those values alone do not prove UFW stop/reload preserves the separate CLARYEL nftables table.

## Last reported checks

```text
PASS: Exact chains
PASS: Chain settings
PASS: Ordered rules
OVERALL: PASS
active
```

The final `active` line was from SSH service status. These checks establish current policy structure and active SSH only.

## Findings and important interpretation correction

1. **Observed:** UFW completed its startup before CLARYEL Guard began its displayed systemd start on this boot.
2. **Observed:** Docker entered active state **before** the Guard systemd service's reported activation. The previous blanket assertion that Guard protected Docker before the initial start is **not supported** by this boot's timestamps.
3. **Observed:** Libvirt entered active state after Guard in this same boot.
4. **UNKNOWN:** Whether Docker's earlier start actually happened without live nftables Guard rules. The Guard loader later explicitly detected an already-existing table, but its creation time is not recorded by these outputs.
5. **Important:** `Requires=claryel-guard.service` and `After=claryel-guard.service` Docker/libvirt drop-ins were installed **after initial service starts**, according to prior owner-reported configuration work. The present boot's chronology therefore cannot validate the subsequently installed dependency configuration. It must be verified on a later controlled restart/reboot with an appropriate recovery path.
6. **UNKNOWN:** Whether UFW could remove, supersede, or affect the Guard through any stop/reload/flush action. Do not infer a universal guarantee from a successful boot-time inspection.
7. **NOT_TESTED:** traffic-level VM isolation, DHCP success after offline Netplan configuration, reboot persistence, simulated UFW failure behavior with updated dependencies.
8. **Security note:** an active oneshot Guard service and the current policy validator do not continuously enforce the continued presence of the nftables table if another process later removes it. Service unit dependencies prevent some start paths when Guard activation fails, but are not an independent always-on firewall integrity monitor.

## Next safe inspection

Read-only check: gather the current `systemctl show` effective relationships, Guard unit enablement and actual systemd unit execution timestamps, plus a scoped journal check for any early manual loading of the Guard. Avoid restarting services, booting VM, or changing firewall rules without owner-approved tests and a rollback.

## Publication safety and provenance

This is an operator-provided transcript without credential values. The author reviewed the outgoing text for known private-key, token and credential-assignment patterns before publication. Automated full Gitleaks and independent gate enforcement were not run and are not claimed.

- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Executed-by:** UNKNOWN (owner-supplied terminal output)
- **Evidence-source:** OWNER_REPORTED
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Published-by:** GitHub connector via shared ViktorIXAI login
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related issues:** #1, #3, #4, #5, #6
