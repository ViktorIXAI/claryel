# P2002 — effective systemd dependencies after Guard drop-ins

- **Checkpoint ID:** CLARYEL-P2002-20261106-SYSTEMD-004
- **Event time (UTC):** UNKNOWN (owner-provided terminal transcript contains no execution wall-clock timestamp)
- **Publication time (UTC):** 2026-11-06T16:25:16Z
- **Target role:** BOX_TEST_HOST
- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Hermes-runtime-machine-ID:** NOT_APPLICABLE (writer is ChatGPT, not a Hermes instance)
- **Target-machine-ID:** P2002
- **Deployment-alias:** P2002-LAB
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Executed-by:** UNKNOWN (owner supplied terminal output; no remote shell execution by author)
- **Evidence-source:** OWNER_REPORTED
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** explicit owner authorization for the temporary P2002 publication window, 2026-11-06
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Operation status:** EXECUTED (read-only inspection commands supplied by owner; no host mutation in this checkpoint)
- **Command exit codes:** UNKNOWN (individual shell exit statuses not recorded)
- **Related checkpoint:** [observed activation chronology](2026-11-06-systemd-activation-order.md)
- **Related issues:** #1, #3, #4, #5, #6

## Scope and purpose

Record the owner's follow-up read-only inspection of current systemd enablement and effective dependencies after installing the Guard dependency drop-ins. This is configuration evidence only; it is not a reboot, restart, packet-level isolation, or firewall persistence test.

## Commands supplied by owner

```bash
for unit in ufw.service claryel-guard.service docker.service libvirtd.service; do
  systemctl is-enabled "$unit" || true
done

for unit in docker.service libvirtd.service claryel-cloud-network.service; do
  systemctl show --no-pager "$unit" -p Requires -p After -p BindsTo
done

systemctl show --no-pager claryel-guard.service \
  -p Requires -p Wants -p Before -p After

sudo virsh -c qemu:///system domstate claryel-cloud-hermes
sudo /usr/local/sbin/claryel-guard-verify
systemctl is-active ssh
```

## Owner-provided observed output

### Enablement

```text
ufw.service: enabled
claryel-guard.service: enabled
docker.service: enabled
libvirtd.service: enabled
```

### Docker effective dependencies

```text
Requires=claryel-guard.service docker.socket system.slice sysinit.target
BindsTo=
After=firewalld.service containerd.service docker.socket network-online.target systemd-journald.socket claryel-guard.service system.slice sysinit.target basic.target
```

### Libvirt effective dependencies

```text
Requires=sysinit.target virtlogd.socket claryel-guard.service system.slice
BindsTo=
After=network.target run-qemu.mount remote-fs.target systemd-journald.socket claryel-guard.service system.slice apparmor.service virtlockd.socket libvirtd-admin.socket virtlogd.socket xencommons.service dbus.service libvirtd-ro.socket libvirtd.socket systemd-machined.service basic.target qemu-kvm.service iscsid.service sysinit.target
```

### Controlled network service effective dependencies

```text
Requires=system.slice sysinit.target claryel-guard.service
BindsTo=
After=system.slice basic.target systemd-journald.socket sysinit.target claryel-guard.service libvirtd.service
```

### Guard effective ordering

```text
Requires=system.slice sysinit.target
Wants=
Before=docker.service libvirtd.service shutdown.target multi-user.target claryel-cloud-network.service
After=systemd-journald.socket sysinit.target basic.target system.slice local-fs.target
```

### Runtime checks

```text
VM state: shut off
PASS: Exact chains
PASS: Chain settings
PASS: Ordered rules
OVERALL: PASS
SSH: active
```

## Findings

1. **PASS — enablement configuration:** UFW, CLARYEL Guard, Docker and libvirtd are all reported enabled.
2. **PASS — Docker start dependency configuration:** effective properties contain both `Requires=claryel-guard.service` and `After=claryel-guard.service`.
3. **PASS — Libvirt start dependency configuration:** effective properties contain both `Requires=claryel-guard.service` and `After=claryel-guard.service`.
4. **PASS — controlled network unit ordering:** it requires Guard and is ordered after Guard and libvirtd.
5. **PASS — Guard ordering declarations:** Guard is ordered before Docker, libvirtd and the controlled network service.
6. **PASS — current reported state:** VM is shut off, the exact nftables policy verifier passes, and SSH is active.
7. **NOT_TESTED — boot execution:** these effective unit properties do not prove that the next boot will complete in this order or that the Guard rules will survive all firewall-management operations.
8. **NOT_TESTED — runtime dependency loss:** Docker and libvirtd show empty `BindsTo=`. This checkpoint does not test behavior if Guard becomes inactive or its nftables table is deleted after dependents have started. A separate failure-mode test and design review are needed; do not assume `Requires=` is a continuous firewall integrity monitor.
9. **NOT_TESTED — UFW interaction:** UFW is enabled and starts independently of the Guard. The output does not establish effects of UFW stop, restart, reload, or flush on the CLARYEL table.
10. **NOT_TESTED — actual isolation:** no guest DHCP, packet-flow, IPv4/IPv6 leak, or reboot-persistence test was run in this checkpoint.

## Interpretation

The currently loaded systemd dependency graph is now configured to gate Docker and libvirt startup on successful Guard activation. This corrects the dependency configuration observed in the earlier installation work. It does not retroactively validate the earlier boot chronology, when Docker was reported active before the Guard service's recorded activation.

A future controlled boot test is still required, with an owner-approved recovery path and evidence capture. No reboot, service restart, VM start, network mutation, or firewall mutation was performed as part of this checkpoint.

## Publication and evidence limitations

The shell transcript is owner-reported. The assistant did not access P2002 directly. The outgoing text was manually reviewed for known credential/token/private-key patterns; no matches were found in the prepared text. Full Gitleaks execution and a fail-closed prepublication gate were **NOT_TESTED**. Therefore issue #1's acceptance criteria remain open; this manual review is not a substitute for the required gate.

- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Executed-by:** UNKNOWN (owner-supplied terminal output)
- **Evidence-source:** OWNER_REPORTED
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** owner-authorized temporary publication window
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
