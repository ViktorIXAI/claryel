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