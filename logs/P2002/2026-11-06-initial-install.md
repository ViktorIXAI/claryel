# P2002 installation log — 2026-11-06

Status: owner-reported observations, not independently verified by remote machine inspection. Public sanitized narrative, no addresses, credentials or serials.

## Installation
- Firmware factory reset; 64 GB USB installer retained in the host for planned recovery.
- Ubuntu Server 24.04 LTS, English, minimal installation; skipped Ubuntu Pro and optional packages; enabled OpenSSH without importing keys.
- Wired DHCP via router; Wi-Fi hardware unused.
- Manual partitioning: planned 400 GB unencrypted ext4 root on approximately 2 TB internal drive; actual layout remains unverified by host commands.
- Installer reported completion and requested reboot. USB left connected.

## Post-install graphics
- Integrated graphics output showed BIOS after reset, but Ubuntu installer used discrete AI compute GPU video; monitor was moved temporarily.
- After system installation and shutdown, monitor attached to integrated motherboard video and dummy-display adapter attached to discrete GPU.
- Host booted but display connected to motherboard remained blank. Console routing to discrete GPU is a hypothesis, not yet proven.
- Requirement for all future installations/recovery: no user monitor cable switching. This acceptance test currently fails.

## Network and SSH
- Original installer DHCP address was not noted.
- Router UI showed alternating previous/current device names and different IP addresses. Router cause is suspected but unconfirmed; no router-side changes planned now.
- One address worked and SSH connectivity was stable. User decided to leave hardware in its current state and continue remotely.

## Checkpoint and pending evidence
- Owner-reported PASS: Ubuntu boot and stable LAN SSH.
- Owner-reported FAIL: no visible local Linux console via integrated GPU.
- UNKNOWN: current DHCP lease and identity, exact root filesystem, SSH hardening, graphics driver/DRM ownership, GPU compute, boot order, firewall and virtualization.
- Next action: read-only diagnostics over existing SSH, redacted before publication. Do not disable/blacklist GPU drivers or change network/firewall until the initial inventory is reviewed.
- Do not publish IP addresses, MAC addresses, usernames, router identifiers, credentials or raw unreviewed command outputs.

## SSH key incident and verified remediation (owner-reported, 2026-11-06)

- During setup, the operator unintentionally generated a client authentication key on the target P2002 itself because commands were entered in the wrong terminal window. This was a human-execution-context error, not evidence of a cryptographic defect.
- With AI-assisted guidance, the operator compared ED25519 public-key fingerprints from the server and the intended ASUS administrative workstation. The records showed two distinct keys with identical human-readable comments. The ASUS key was positively matched to its public key on the workstation.
- The operator removed the accidental P2002 private/public key pair and only its matching entry in authorized_keys. Subsequent authorized_keys fingerprint output contained only the confirmed ASUS workstation key. No private key material is recorded here.
- Subsequent login from ASUS to P2002 succeeded. The effective OpenSSH configuration was independently printed by sshd -T and showed pubkeyauthentication=yes, passwordauthentication=no, kbdinteractiveauthentication=no, permitrootlogin=no, authenticationmethods=publickey.
- Mac Studio backup access is **explicitly not granted** at this stage because another autonomous AI agent operates on that computer. The single authorized SSH public key is for the ASUS workstation. Network-level source filtering has not yet been proven.
- **Lessons learned:** label every command with its execution machine; run read-only preflight and compare expected state; use small changes followed by checks; preserve a safe rollback route; redact operational output before public publication. AI guidance can make mistakes and must not be considered intrinsically trustworthy. The operator remains the authorizer for destructive and access-changing commands.
- Next step: perform a read-only host/security/virtualization inventory, then controlled software updates and hypervisor/container setup after review. The end-to-end deployment is not yet complete.

## Security and virtualization preflight (owner-provided terminal output)

- Confirmed account/hostname correspond to intended P2002 session (identity details sanitized here).
- UFW reports **inactive**; no firewall hardening has been claimed or executed at this checkpoint.
- Effective SSH configuration: root login disabled; public-key authentication enabled; password and keyboard-interactive authentication disabled; public-key-only authentication required.
- **/dev/kvm exists**, with kvm and kvm_intel kernel modules loaded. KVM-capable hardware/kernel confirmed, but hypervisor userspace not yet installed or tested.
- Package query returned no installed qemu-kvm, libvirt-daemon-system, docker-ce, docker.io or containerd.io in the queried package set. This does not prove there are no other VM/container runtimes.
- Multiple Ubuntu 24.04 security/maintenance updates are pending. No updates or reboot performed for this report; do not upgrade to Ubuntu 26.04 as part of this checkpoint.
- Next safe step: verify source address of the current ASUS SSH connection, pre-authorize that exact address for port 22 on both relevant protocol families, then enable firewall with rollback and new-connection verification. Do not configure source-specific firewall solely from earlier IP claims. Docker and libvirt network behavior will need subsequent security review.
