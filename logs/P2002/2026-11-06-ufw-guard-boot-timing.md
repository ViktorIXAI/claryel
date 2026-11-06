# P2002 — UFW/Guard startup timing and nftables bridge inspection

- **Checkpoint ID:** CLARYEL-P2002-20261106-UFW-002
- **Event time (UTC):** UNKNOWN — terminal transcript contained no execution timestamp.
- **Publication date:** 2026-11-06; actual GitHub commit time is publication metadata, not an execution time.
- **Target-machine-ID:** P2002 (owner-authorized public alias)
- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Deployment-alias:** NOT_APPLICABLE
- **Hermes-runtime-machine-ID:** NOT_APPLICABLE — the writer here is not Hermes.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Executed-by:** UNKNOWN (operator-supplied terminal transcript; ChatGPT did not execute P2002 commands)
- **Evidence-source:** OWNER_REPORTED
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** owner-authorized temporary 24-hour technical-log window (2026-11-06)
- **Published-by:** GitHub connector using the shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Operation status:** EXECUTED (owner-reported read-only diagnostics only)
- **Exit codes:** UNKNOWN; the transcript did not record shell `$?` values
- **Related checkpoint:** CLARYEL-P2002-20261106-UFW-001
- **Related issues:** #1, #3, #4, #5, #6

## Diagnostic commands — operator-provided

```bash
systemctl cat claryel-guard.service
systemd-analyze critical-chain ufw.service
systemd-analyze critical-chain claryel-guard.service
sudo sed -n '1,240p' /usr/lib/ufw/ufw-init
sudo sed -n '110,190p' /usr/lib/ufw/ufw-init-functions
sudo nft list tables
sudo nft list tables bridge
lsmod | grep -E 'br_netfilter|bridge' || true
sudo /usr/local/sbin/claryel-guard-verify
systemctl is-active ssh
```

## Guard unit — displayed configuration

```ini
# /etc/systemd/system/claryel-guard.service
[Unit]
Description=CLARYEL isolated VM firewall guard
After=local-fs.target
Before=docker.service libvirtd.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/local/sbin/claryel-guard-load
ExecReload=/usr/local/sbin/claryel-guard-load

[Install]
WantedBy=multi-user.target
```

Earlier owner-reported output showed additional Docker/libvirt drop-in units with `Requires=claryel-guard.service` and `After=claryel-guard.service`. These drop-ins are not independently re-read by this diagnostic block.

## UFW critical-chain output

```text
ufw.service +2.741s
└─local-fs.target @613ms
  └─boot-efi.mount @570ms +39ms
```

## Guard critical-chain output

```text
claryel-guard.service +40ms
└─basic.target @783ms
  └─sockets.target @783ms
    └─sysinit.target @763ms
```

The supplied terminal displayed longer descendant paths in both chains. One terminal line was truncated by its pager and contained a filesystem UUID. This public checkpoint omits that irrelevant identifier.

**Interpretation:** `critical-chain` displays activation times on a dependency chain, not a complete chronological comparison of unit start/end events. Neither displayed service line included a direct absolute activation timestamp after `@`. The excerpts do **not** prove UFW completed before Guard or the reverse. Use timestamped `journalctl` entries and explicit `systemctl show` timestamps to determine observed order, then assess whether there is a guaranteed unit ordering dependency. A successful ordering observation on one boot is not a guarantee for future boots.

## UFW initialization script — displayed behavior

The owner displayed the top of `/usr/lib/ufw/ufw-init`. Relevant statements:

```sh
set -e

. "${rootdir}/etc/ufw/ufw.conf"
if [ "$1" = "start" ] && [ "$2" = "quiet" ] && [ "$ENABLED" = "no" ]; then
    exit 0
fi

if [ -s "${rootdir}/lib/ufw/ufw-init-functions" ]; then
    . "${rootdir}/lib/ufw/ufw-init-functions"
else
    echo "Could not find ${rootdir}/lib/ufw/ufw-init-functions (aborting)"
    exit 1
fi
```

The case statement dispatches the following actions:

```sh
start)      # calls ufw_start; preserves return status
    ;;
stop)       # calls ufw_stop
    ;;
force-stop) # calls ufw_stop --force
    ;;
restart|force-reload) # calls ufw_reload
    ;;
flush-all)  # calls optional before.init flush-all, flush_builtins,
            # then optional after.init flush-all
    ;;
```

These are **annotated excerpts** of the displayed script, not an independently executable reproduction of its full source.

The `flush-all` comment in the source explicitly warns that it flushes built-in chains, deletes non-built-in chains and resets policy to ACCEPT. The actual effect on the separate `inet claryel_guard` table was **not tested**. Do not infer that the command was run; the script was only read.

## UFW initialization functions — displayed behavior

From the displayed `/usr/lib/ufw/ufw-init-functions` range:

```sh
if [ "$ENABLED" = "yes" ] || [ "$ENABLED" = "YES" ]; then
    if iptables -L ufw-user-input -n >/dev/null 2>&1 ; then
        echo "Firewall already started, use 'force-reload'"
        return 0
    fi
    for m in $IPT_MODULES
    do
        modprobe $m || true
    done

    if [ "$MANAGE_BUILTINS" = "yes" ]; then
        flush_builtins
    fi
    # before.init hook and subsequent iptables/ip6tables processing omitted
fi
```

The supplied range also showed the IPv6-disabled branch calling `delete_chains 6` and `ip6tables-restore`, and a later loop calling `delete_chains $type` when UFW logging chains exist.

**Interpretation:** UFW can alter the iptables-compatible rule sets under certain settings. A search or source excerpt alone does not establish that its reload/stop operations preserve all independent nftables objects; neither behavior was invoked in this checkpoint.

## NFT tables — owner-supplied list

```text
table ip filter
table ip6 filter
table ip nat
table ip mangle
table ip6 nat
table ip6 mangle
table inet claryel_guard
```

`nft list tables bridge` returned no table lines in the supplied output.

`lsmod | grep -E 'br_netfilter|bridge'` returned:

```text
bridge                425984  0
stp                    12288  1 bridge
llc                    16384  2 bridge,stp
```

This establishes that the `bridge` module was listed and `br_netfilter` was not listed by this command at that moment. It does not by itself demonstrate how all Ethernet bridge paths are filtered, nor whether another bridge mechanism is in effect.

## Final checks — owner-reported

```text
PASS: Exact chains
PASS: Chain settings
PASS: Ordered rules
OVERALL: PASS
active
```

The final `active` line was emitted by `systemctl is-active ssh`. The Guard validator's PASS proves only its exact expected table structure at that point.

## Explicit limitations and outstanding checks

- **UNKNOWN:** actual UFW-versus-Guard completion order. The two `critical-chain` outputs do not prove it.
- **NOT_TESTED:** UFW restart, stop, force-reload and flush-all impact on the live Guard. These commands must not be run casually on a remotely administered host.
- **NOT_TESTED:** firewall rule persistence after a fresh P2002 reboot.
- **NOT_TESTED:** live VM packet-level egress denial, DNS bypass, IPv6 behavior and L2 bridge isolation.
- **NOT_TESTED:** successful DHCP and host-to-guest SSH after the guest's persistent-interface configuration changes.
- **UNKNOWN:** whether UFW's boot-time changes or late service activation could ever erase or supersede Guard rules. Current successful validation does not establish an invariant.
- **NOT_APPLICABLE:** no new service, VM, network, or firewall state changes in this checkpoint.

## Next safe step — no host changes

Collect read-only activation timestamps from systemd and selected UFW configuration flags, then compare actual event order and supported dependency guarantees before proposing changes. Do not run `ufw reload`, `ufw disable`, `flush-all`, `nft flush ruleset`, reboot the host, or boot the guest in this diagnostic phase.

## Contribution provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: NOT_APPLICABLE
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: UNKNOWN — owner-provided terminal output
- Evidence-source: OWNER_REPORTED
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: explicit owner authorization for 24-hour technical-log publication
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
