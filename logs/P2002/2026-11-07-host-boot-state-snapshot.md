# P2002 — Host boot and Cloud Hermes VM state snapshot

- **Checkpoint ID:** CLARYEL-P2002-20261107-HOST-BOOT-SNAPSHOT-001
- **Event timestamp:** 2026-11-07T03:24:14+00:00 (host output)
- **Target-machine-ID:** P2002
- **Operation:** READ_ONLY_DIAGNOSTICS
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner on P2002; ChatGPT did not access the host or execute these commands
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Deployment-alias:** P2002-LAB
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (read-only diagnostic request)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoints:** CLARYEL-P2002-20261107-CLOUD-AUTOSTART-001; CLARYEL-P2002-20261107-GUEST-RUNTIME-DISCOVERY-001

## Command execution note

The owner ran the grouped read-only snapshot on P2002. The first sudo password attempt was rejected; the second succeeded. The command block completed and emitted the final marker. No configuration changes were requested or made by this diagnostic.

## Observed host state

- Kernel: `7.0.0-42-generic`.
- `claryel-guard.service`: loaded, active (`exited`), enabled.
- `docker.service`: loaded, active (`running`), enabled.
- `libvirtd.service`: loaded, active (`running`), enabled.
- `virtqemud.service`: not found. The running libvirt stack is using the legacy monolithic `libvirtd` service; no failure is inferred from the absent modular daemon unit.
- The displayed systemd properties show Guard ordered before Docker, libvirt and multi-user startup. Docker and libvirt list Guard in `Requires=`; the displayed `After=` value for Docker/libvirt was truncated by terminal formatting, so the complete effective ordering is not established by this snapshot alone.
- Related active services included `claryel-proxy-firewall.service`, `libvirt-guests.service`, `virtlockd.service` and `virtlogd.service`.

## Observed libvirt state

For `claryel-cloud-admin`:
- Active: yes.
- Persistent: yes.
- Autostart: yes.
- Bridge: `virbr240`.

For `claryel-cloud-hermes`:
- State: running.
- Persistent: yes.
- Autostart: enabled.
- AppArmor security label: enforcing.

The libvirt DHCP lease query reported guest IPv4 `172.30.240.12/24` on `vnet0`. The MAC address was omitted from this public checkpoint.

## Observed Guard table

The `inet claryel_guard` nftables table was present at the time of the snapshot. The displayed `guest_input` and `guest_forward` chains had explicit drops for traffic entering/leaving the isolated guest bridge; the Strata egress bridge had a narrow proxy allowance and explicit drops for other traffic. The base-chain policies displayed as `accept`; this record does not interpret that alone as proof of end-to-end isolation or fail-closed behavior.

## Result and limitations

- **PASS (snapshot only):** Guard unit is active and enabled.
- **PASS (snapshot only):** Docker and legacy libvirt services are active and enabled.
- **PASS (snapshot only):** intended libvirt network is active, persistent and configured for autostart.
- **PASS (snapshot only):** Cloud Hermes VM is running, persistent and configured for autostart.
- **PASS (snapshot only):** guest DHCP lease is present at `172.30.240.12/24`.
- **OBSERVED:** Guard nftables table exists and contains the displayed bridge restrictions.
- **NOT_TESTED:** actual cold-boot ordering, reboot behavior, unexpected power-loss recovery, continuous Guard enforcement, guest Hermes process/service startup, guest-to-Strata integration, and end-to-end isolation.
- **NO_CHANGES:** this was a read-only host snapshot.

This checkpoint confirms the current autostart flags and running state; it does **not** satisfy issue #10's controlled reboot acceptance criterion. The guest-side Hermes startup mechanism remains the next installation task. Preserve the existing Hermes v0.23.0 installation and Grok authorization; do not reinstall or reauthorize.

## Next action

Continue with a grouped read-only inventory inside the existing guest to identify the already-installed Hermes CLI's supported persistent/gateway mode and any existing supervisor/container/service startup mechanism. Do not display configuration contents, `.env` values, OAuth tokens, API keys or other secrets. Once the actual launch mechanism is established, configure only that existing installation for boot startup and validate it before any controlled host reboot.

