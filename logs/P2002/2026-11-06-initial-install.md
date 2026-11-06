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

## Pre-UFW activation client and interface validation

Operator output showed the SSH client is connected over wired Ethernet to TCP port 22. The client IPv4 address was identified locally but is intentionally omitted from this public log. Host UFW has IPv6 support enabled but reports inactive. Wi-Fi is down; the host also has globally routed IPv6, therefore ingress rules must be reviewed for both address families. Next staged operation: default-deny incoming, default-allow outgoing, then allow inbound TCP/22 on the named Ethernet interface *only from the active administrative workstation IPv4*. Validate the rules while UFW remains inactive. Only after review, activate with controlled rollback and establish a fresh authenticated connection before closing the prior session. **No firewall activation is claimed here.**

## Permanent logging directive

Owner requires public, sanitized documentation of each configuration action, outcome, error and correction on P2002 and future CLARYEL boxes, for independent AI review. Policy: policies/INSTALLATION-AUDIT-LOGGING.md. At this checkpoint, the UFW SSH allow rule has been verified but firewall activation remains unconfirmed.

## Ubuntu package update checkpoint (owner terminal report)

- Owner ran system package upgrades within Ubuntu 24.04 LTS. Afterwards apt reported no further upgradable packages in the displayed query.
- The system reported a required reboot. needrestart reported service restarts deferred and existing SSH sessions using older binaries; no containers or running VM guests required updates.
- OpenSSH remained active. UFW remained active with default-deny incoming and only the previously authorized administrative workstation SSH source over the wired interface; IPv6 ingress remains denied by default.
- **Reboot and post-reboot verification have not yet been reported.** Because local integrated video console remains unavailable, a fresh SSH login from the authorized workstation is required immediately before reboot. After reboot verify new kernel, uptime/boot, SSH, firewall, network and reboot-required flag.
- No upgrade to Ubuntu 26.04 was requested or performed, and no successful reboot or service deployment is claimed.

## Verified reboot checkpoint following package updates (owner evidence, 2026-11-06)

- The host rebooted successfully after Ubuntu 24.04 updates and a fresh SSH connection from the authorized administrative ASUS was established.
- Operating system now reports Ubuntu Server **24.04.5 LTS**, kernel **6.8.0-149-generic**.
- systemd reports `running`, the wired network is UP, wireless is DOWN.
- OpenSSH service is active. UFW remains active across reboot, default-deny incoming, default-allow outgoing, only wired interface inbound TCP/22 from the designated administrative client address. **Actual IPs intentionally omitted from public log**.
- Update query: no pending updates shown. Reboot-required marker: absent.
- This confirms base host SSH/firewall persistence after reboot, **not** completed application, network isolation, or compute GPU acceptance.
- Next: install vetted QEMU/KVM and libvirt user-space packages without configuring a public bridge or starting cloud VM; check libvirt virtual networks before accepting any internet-enabled guest.

## KVM/libvirt installation checkpoint (owner output)

- `kvm-ok` reports that /dev/kvm exists and KVM acceleration is usable.
- `virsh -c qemu:///system list --all` returned no virtual machines.
- `virsh -c qemu:///system net-list --all` showed a pre-created libvirt `default` network in the **active** state with autostart enabled. No claim is made that this NAT network satisfies CLARYEL isolation.
- SSH stayed active and UFW stayed active with default-deny incoming, default-allow outgoing and deny routed, and a single administrative source rule. Exact addresses are omitted from this public log.
- Proposed next operation, **not yet confirmed executed**: disable autostart of libvirt default network; destroy its current active instance (not undefine); verify network is inactive/autostart=no and SSH/UFW remain active. Future cloud VM network must be independently designed and tested.

## Libvirt default network disabled (owner-verified)

The operator executed the planned libvirt commands and reported successful completion. The libvirt `default` network is now **inactive**, **autostart=no**, and **persistent=yes** (configuration retained); no VM was running at the earlier inventory. SSH remains active; UFW remains active with default-deny incoming, default-allow outgoing, deny routed, and the single authorized workstation SSH rule. This confirms the local libvirt default-network state only, **not** network isolation of future guest VMs or Docker containers. Next milestone: review container runtime installation and networking before any guest or container external exposure. No sensitive host addresses are published.

## Docker installation and firewall preflight (owner terminal evidence)

- Ubuntu-packaged Docker Engine 29.1.3 and Docker Compose 2.40.3 installed and operational; daemon active, no containers. Standard bridge, host and none networks exist.
- Host IPv4 forwarding is enabled; IPv6 forwarding is disabled. UFW remains active with deny incoming and deny routed.
- No daemon.json exists. Docker default bridge configuration currently enables inter-container communication (ICC) and masquerading; published ports default to binding on all IPv4 addresses. docker0 is currently without carrier.
- iptables v1.8.10 uses nf_tables compatibility backend. DOCKER-USER chain exists; top FORWARD rules jump into DOCKER-USER and DOCKER-FORWARD before UFW chains, and FORWARD default policy is DROP. These observations do not prove container traffic isolation or public-port restrictions.
- The host listens on SSH over IPv4 and IPv6, but UFW currently limits permitted inbound SSH to the designated administrative workstation IPv4. Loopback DNS and an additional loopback listener were also observed. Exact private host addresses and generated IDs deliberately omitted.
- No new containers, exposed ports, traffic policies or firewall modifications have yet been performed. Next: read-only review of Docker forwarding/NAT rules, then a narrowly scoped security configuration and explicit tests before applications are deployed.

## Isolated Docker test network created (owner terminal evidence)

- An empty test Docker bridge named `claryel-test-isolated` was successfully created using `--internal` and `com.docker.network.bridge.gateway_mode_ipv4=isolated`.
- Docker inspection confirmed `Internal=true` and the `isolated` IPv4 gateway mode option. No containers were present.
- SSH and UFW remained active with the existing restrictive host rules.
- **Network isolation is not yet empirically proven:** no test container, external reachability probe or inter-zone denial test has run. Next validation should run a minimal unprivileged disposable container, check external, LAN and host reachability, and report observed exit status for each probe. Avoid publishing live IPs and machine identifiers.

## First empirical Docker network isolation probe (owner-supplied output)

- Downloaded a pinned BusyBox 1.36.1 test image on the host before the probe.
- Launched an ephemeral container on the dedicated internal bridge using all Linux capabilities dropped and no-new-privileges; container automatically removed on exit.
- Inside the container, the route table showed only the directly attached bridge subnet, with **no default route**.
- Reported TCP connection probes to an external public endpoint, a LAN gateway endpoint and host SSH each returned a blocked outcome, labeled PASS by the test script. Those probes target specific ports and cannot prove universal network isolation.
- Post-test evidence: no containers remained; SSH and UFW stayed active with the previously authorized administrative source restriction.
- The pasted shell transcript contained visible line corruption near the final probe command, although output included all three probe outcomes. Treat results as **preliminary PASS with a reproducibility caveat**. Future tests must record independent exit codes and include DNS/UDP, lateral traffic, host reachability and multiple attached networks.
- No application service, agent or private data was deployed during this test.

## Two-zone Docker network isolation test (owner-provided results)

- Created isolated internal bridges Zone A and Zone B with distinct IPv4 subnets and isolated gateway mode.
- A disposable BusyBox HTTP server was started on Zone A without published host ports or additional capabilities.
- **Positive control PASS:** a temporary client attached to Zone A retrieved the known `CLARYEL_OK` response from that server.
- **Negative control PASS (tested direction only):** a temporary client attached solely to Zone B received `Network is unreachable` when attempting to connect to the Zone A server on TCP 8080.
- This confirms reachability within Zone A and failure of the tested B-to-A connection, not a comprehensive demonstration of security isolation. Reverse direction, other protocols, multi-homed containers, and host-boundary behavior remain untested.
- The persistent Zone A test server has **not yet been confirmed removed** at this checkpoint. Clean it up and verify the container inventory before proceeding. The exact dynamic container IP is omitted from the public log.

## Cleanup after two-zone test (owner-verified)

- Operator removed the temporary Zone A HTTP server using Docker; the removal command returned its expected container name.
- `docker ps -a` showed zero remaining containers. The two isolated internal test networks remain configured for further reverse-direction testing.
- SSH and UFW remained active with default-deny ingress and restricted authorized workstation SSH access.
- Next verification is the reverse A-to-B connectivity attempt with a positive control inside Zone B; no result for that direction is claimed yet.

## Reverse-direction Docker isolation test (operator output)

- A temporary HTTP server was started on isolated Zone B, without publishing host ports.
- **Positive control passed:** an ephemeral client in Zone B retrieved `CLARYEL_B_OK` over HTTP from the server.
- **Negative control passed:** an ephemeral client on Zone A could not connect to the Zone B HTTP endpoint. The reported error was `Network is unreachable`.
- Combined with the earlier A-zone positive control and B-to-A negative test, this shows successful TCP/HTTP checks in both zones and blocked tested cross-zone connections in each direction.
- Scope limitation: this is not proof of isolation for a dual-homed container, privileged container, host service, additional route or alternate protocol.
- As of this report, **removal of the temporary Zone B server has not been confirmed**. Cleanup and inventory checks are required before further tests.

## Dual-homed test intermediary creation (owner-provided terminal output)

- Operator started a temporary BusyBox container with all Linux capabilities dropped and no-new-privileges, initially attached to isolated Zone A, then connected it to isolated Zone B.
- Docker inspection and `ip addr` showed two live Ethernet interfaces, one in each distinct Docker isolated subnet. `ip route` showed only the two directly connected subnet routes and no default gateway.
- SSH and UFW remained active under existing host access restrictions. There were no privileged container flags or published host ports in this step.
- Crucial security limitation: while the kernel has no default route, a dual-network application may still receive and emit messages on both attached networks and thereby bridge them at application level. No positive application-reachability probe or forwarding bypass has yet been recorded.
- Next: run test servers separately in zones A/B, check each from the intermediary, and clean up ephemeral containers. Zone addresses are intentionally excluded from the public log.

## Dual-network test completed and cleaned up (owner-verified)

- A temporary, unprivileged, two-network-attached test container successfully fetched the expected HTTP test responses from services on each isolated Docker network. This shows reachability to both networks from the intermediary, but not kernel packet forwarding or production gateway security.
- All three temporary test containers (the intermediary and two HTTP servers) were subsequently removed. Docker reported no remaining containers. Two isolated Docker test networks were retained.
- Libvirt reported no VMs. Its standard default network remained inactive with autostart disabled.
- SSH and UFW remained active with the previously configured restricted administrative connection.
- Architecture reaffirmed: internet-connected Cloud Hermes will run in its own KVM/QEMU guest, not in a Docker container on P2002. Private services can use Docker; the Security Gateway crossing remains to be designed, implemented, and tested.
- Next action: inspect available host resources, VM tooling, image sources and storage layout before allocating VM storage or enabling guest networking.
