# P2002 — Cloud Hermes network and VM autostart enabled

- **Checkpoint ID:** CLARYEL-P2002-20261107-CLOUD-AUTOSTART-001
- **Event date:** 2026-11-07 (owner-supplied terminal transcript; exact wall-clock time not present)
- **Target-machine-ID:** P2002
- **VM:** `claryel-cloud-hermes`
- **Libvirt network:** `claryel-cloud-admin`
- **Evidence source:** OWNER_REPORTED
- **Executed by:** Owner on P2002; commands and output supplied in conversation
- **Publication actor:** ChatGPT GitHub connector
- **Operation status:** EXECUTED

## Commands executed

```bash
virsh net-autostart claryel-cloud-admin
virsh autostart claryel-cloud-hermes
```

## Owner-provided output

```text
Network claryel-cloud-admin marked as autostarted
Domain 'claryel-cloud-hermes' marked as autostarted

Name:           claryel-cloud-admin
Active:         yes
Autostart:      yes
Name:           claryel-cloud-hermes
State:          running
Autostart:      enable
```

## Result

- **PASS — libvirt network autostart:** enabled.
- **PASS — VM autostart:** enabled.
- **PASS — current runtime:** network active and VM running at the time of the supplied output.
- The network and VM are configured to start automatically after host/libvirt startup, subject to the already documented Guard-before-libvirt dependency.
- No reboot was performed in this checkpoint.
- No guest IP, DHCP lease, guest SSH, or guest application readiness claim is made from this output.

## Next implementation phase

Proceed with the Cloud Hermes guest installation using the previously established Ubuntu Server 24.04.5 guest and existing controlled access path. Keep the current isolated libvirt network and Guard policy intact. Do not add NAT/default internet routing or weaken host firewall isolation merely to simplify installation. Group the next meaningful setup operations and perform one consolidated validation after the batch.

## Publication safety

This checkpoint contains no credentials, private keys, tokens, or guest authorized-key material.

## Owner correction — existing guest Hermes installation (2026-11-07)

**This addendum supersedes the preceding section's proposed next step to install Hermes. Do not reinstall or replace the existing guest agent.**

The owner explicitly corrected the assistant's assumption and reports that:
- Hermes is already installed inside the existing `claryel-cloud-hermes` VM.
- The guest Hermes instance is connected to Grok and provider authorization has already been completed.
- The existing agent has already passed response tests.

**Evidence classification: OWNER_REPORTED.** The public repository snapshot reviewed for this correction did not contain the original installation commands, Hermes version, guest service/unit name, Grok authorization method, or raw response-test transcript. No credential, token, private key, or account detail is requested or published. These owner-reported accomplishments must not be presented as independently verified until the corresponding sanitized evidence is recovered.

### Correct next operational objective

1. Preserve the existing guest, Hermes installation, Grok authorization and agent state.
2. Inspect the guest's existing Hermes process/service and its autostart enablement; do not reinstall.
3. If autostart is not enabled, enable the existing unit using its actual discovered name; do not guess a unit name.
4. Confirm Hermes starts and remains available after VM startup.
5. Continue with the planned Hermes-to-local-inference integration using the already working private Strata API, with synthetic data and the existing network/Guard boundaries.

No guest command or service change was performed by this documentation correction.

- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Executed-by:** NOT_APPLICABLE (documentation only; owner supplied the correction)
- **Evidence-source:** OWNER_REPORTED + TOOL_OBSERVED repository review
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED


## Root-cause clarification — why the VM did not start after the 2026-11-06 host reboot

The initial installation direction is preserved in [issue #8](https://github.com/ViktorIXAI/claryel/issues/8): keep both `claryel-cloud-admin` network autostart and `claryel-cloud-hermes` VM autostart **off initially** while bringing up the installation under operator control. This was an intentional temporary configuration, not a KVM or qcow2 failure.

The subsequent owner-supplied evidence recorded in [issue #10](https://github.com/ViktorIXAI/claryel/issues/10) shows:
- P2002 rebooted at approximately 2026-11-06 18:26 UTC.
- The VM remained defined and persistent; its qcow2 disk remained present.
- KVM modules and `/dev/kvm` were available.
- The VM log recorded a clean shutdown at 18:25:37 UTC after SIGTERM from `libvirtd`, immediately before the host reboot.
- `virsh dominfo` reported `Autostart: disable`.
- The libvirt network autostart was also disabled at the later 2026-11-07 owner-supplied current-state checkpoint.

**Root cause:** the VM was shut down normally as part of host shutdown and was not started again because its libvirt autostart flag had deliberately been left disabled by the initial installation plan. The network was likewise not configured to auto-start. The available evidence does not indicate a VM disk failure, missing VM definition, or KVM failure.

**Responsibility/sequence:** the initial plan recorded the owner's then-current instruction to leave autostart off. The later owner direction changed that requirement, but the flags were not enabled before the host reboot. The installation coordinator should have tracked that temporary state and completed the transition when the direction changed. This is a process/configuration omission, not an unexplained external failure.

The owner later executed the two commands recorded above:
```bash
virsh net-autostart claryel-cloud-admin
virsh autostart claryel-cloud-hermes
```
Both flags subsequently appeared enabled in the owner-supplied host snapshot. **A controlled P2002 reboot has not yet been performed**, so automatic recovery is configured but not proven.

This is separate from guest Hermes autostart: earlier guest diagnostics found no Hermes system/user service or process and `Linger=no`. The later user Gateway unit and `Linger=yes` address that separate layer. See the [PM/Gateway/Grok log](2026-11-07-cloud-hermes-network-pm-repair-gateway.md).

Evidence classification: issue #8/#10 and owner-supplied terminal outputs; no independent host access by this writer.
