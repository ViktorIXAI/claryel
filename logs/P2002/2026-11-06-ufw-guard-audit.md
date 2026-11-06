# P2002 — UFW and CLARYEL Guard diagnostic checkpoint

- **Checkpoint ID:** CLARYEL-P2002-20261106-UFW-001
- **Event time (UTC):** UNKNOWN — terminal output was supplied without an execution timestamp.
- **Publication time (UTC):** Recorded in the GitHub commit metadata; not an execution timestamp.
- **Target role / machine alias:** P2002 host
- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Deployment-alias:** NOT_APPLICABLE
- **Hermes-runtime-machine-ID:** NOT_APPLICABLE — this contribution was prepared by ChatGPT, not by a Hermes instance.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Executed-by:** UNKNOWN — the owner supplied terminal output; ChatGPT did not execute commands on P2002.
- **Evidence-source:** OWNER_REPORTED
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** Explicit owner authorization for the 24-hour P2002 technical-log publication window, 2026-11-06.
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Result scope:** Read-only inspection of UFW, the installed Guard loader/verifier, selected firewall-related files and active systemd units.
- **Host changes in this checkpoint:** None reported. The commands shown are inspection commands.

## 1. UFW systemd unit — owner-supplied output

The unit shown by `systemctl cat ufw.service` was:

```ini
# /usr/lib/systemd/system/ufw.service
[Unit]
Description=Uncomplicated firewall
Documentation=man:ufw(8)
DefaultDependencies=no
Before=network-pre.target
Wants=network-pre.target local-fs.target
After=local-fs.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/lib/ufw/ufw-init start quiet
ExecStop=/usr/lib/ufw/ufw-init stop

[Install]
WantedBy=multi-user.target
```

The corresponding `systemctl show ufw.service` output included:

```
Before=network-pre.target
After=systemd-journald.socket system.slice local-fs.target
```

The ExecStart property was truncated by the terminal pager in the `systemctl show` output; the complete ExecStart and ExecStop above come from `systemctl cat`.

## 2. Installed CLARYEL Guard loader — owner-supplied source

The installed `/usr/local/sbin/claryel-guard-load` content was displayed as:

```bash
#!/bin/bash
set -euo pipefail

RULES=/etc/claryel/firewall.nft
VERIFY=/usr/local/sbin/claryel-guard-verify

if [[ ! -f "$RULES" || ! -x "$VERIFY" ]]; then
    echo "FAIL: Required guard files missing" >&2
    exit 1
fi

/usr/sbin/nft --check --file "$RULES"

if /usr/sbin/nft list table inet claryel_guard \
    >/dev/null 2>&1; then
    echo "Existing guard detected; verifying policy"
else
    echo "Guard absent; loading policy"
    /usr/sbin/nft --file "$RULES"
fi

"$VERIFY"

echo "PASS: CLARYEL guard verified"
```

Interpretation limited to the displayed source:
- The loader checks that the rules file and verifier exist.
- It runs an nftables syntax check against the rules file.
- If the table already exists, it verifies the existing policy; it does not replace a present but incorrect table.
- If the table is absent, it loads the rules file and then runs the verifier.
- A successful loader run verifies the exact policy expected by the installed verifier; it does not establish packet-level isolation by itself.

## 3. Installed CLARYEL Guard verifier — owner-supplied source

The displayed verifier is a Python program that obtains JSON from:

```bash
nft -j list table inet claryel_guard
```

It expects exactly two chains:
- `guest_input`: filter hook `input`, priority `-10`, policy `accept`.
- `guest_forward`: filter hook `forward`, priority `-10`, policy `accept`.

It expects the following ordered rules:
1. `guest_input`: interface `virbr240`, established/related connection state, accept.
2. `guest_input`: interface `virbr240`, UDP source port 68 and destination port 67, accept.
3. `guest_input`: interface `virbr240`, drop.
4. `guest_forward`: input interface `virbr240`, drop.
5. `guest_forward`: output interface `virbr240`, drop.

The verifier prints separate results for exact chain names, chain settings and ordered rules, and exits non-zero if any of those checks fail. It does not inspect other nftables tables/families or prove actual traffic behavior. Earlier owner-supplied terminal outputs in this conversation showed the verifier returning OVERALL PASS; the exact diagnostic block recorded here did not rerun it.

## 4. Search for firewall restore/flush commands

The owner ran a recursive text search for:

```bash
grep -RInE \
  'flush ruleset|nft[[:space:]]+flush|iptables-restore|ip6tables-restore' \
  /etc/ufw /usr/lib/ufw /etc/systemd/system \
  /usr/local/sbin/claryel-*
```

The only displayed match was:

```
/usr/lib/ufw/ufw-init-functions:153:"COMMIT\n" | ip6tables-restore || error="yes"
```

Observed conclusion:
- The search returned one `ip6tables-restore` occurrence in the searched paths.
- No `flush ruleset` or `nft flush` match was displayed in that search.
- This is not proof that no other component, script, binary or path can change nftables state.
- The displayed line is an IPv6 iptables restore operation; it is not evidence of a global `nft flush ruleset`.
- The effect of UFW start, stop or restart on the separate `inet claryel_guard` table has not been tested.

## 5. Active systemd units — owner-supplied output

The displayed filtered `systemctl list-units --all` output included:

```
claryel-cloud-network.service  loaded active exited  CLARYEL Cloud Hermes controlled network
claryel-guard.service         loaded active exited  CLARYEL isolated VM firewall guard
ssh.service                  loaded active running OpenBSD Secure Shell server
ufw.service                  loaded active exited  Uncomplicated firewall
ssh.socket                   loaded active running  OpenBSD Secure Shell server socket
```

A first attempt to read the Guard loader encountered a failed sudo authentication attempt; a subsequent prompt succeeded and the file contents were displayed. No password value was included in the supplied terminal output.

## 6. Findings and unresolved questions

1. UFW has `DefaultDependencies=no`, starts before `network-pre.target`, and has an explicit `ExecStop` that invokes `ufw-init stop`.
2. The Guard loader is designed to verify an existing table or load a missing table. It does not repair a table that exists but fails verification.
3. The verifier checks the exact two-chain policy and ordered rules only. It is not a complete audit of every nftables table, bridge-family rules, iptables compatibility chains, or real traffic.
4. The search found one IPv6 iptables restore operation. Its effect on `inet claryel_guard` has not been tested.
5. The current outputs do not establish an explicit UFW-to-Guard systemd dependency. Actual boot ordering still needs the requested critical-chain/dependency inspection.
6. UFW restart/stop behavior, reboot persistence, live packet-level isolation, IPv6 bypass behavior, and inter-guest L2 isolation remain NOT_TESTED in this checkpoint.
7. Earlier owner-supplied outputs showed Cloud Hermes shut off, the management network active/manual-only, and the persistent VirtIO interface configured. Those facts are not revalidated by the specific UFW diagnostic block recorded here.

## 7. Next safe diagnostic step

Continue with read-only inspection of:
- `systemd-analyze critical-chain ufw.service`;
- `systemd-analyze critical-chain claryel-guard.service`;
- the complete `/usr/lib/ufw/ufw-init` and relevant `ufw-init-functions` implementation;
- the exact Guard unit dependencies.

Do not restart UFW, reload firewall rules, boot the guest, or change network configuration as part of this diagnostic step. Any such operation requires a separate risk assessment and explicit execution authorization.

## Contribution provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: NOT_APPLICABLE
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: UNKNOWN (owner-supplied terminal output; ChatGPT did not execute on P2002)
- Evidence-source: OWNER_REPORTED
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: explicit owner instruction for 24-hour technical-log publication
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related issues: #1, #3, #4, #5, #6
