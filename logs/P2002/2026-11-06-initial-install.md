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

## KVM guest capacity preflight (operator-reported)

- The P2002 host exposes 24 logical CPUs and approximately 61 GiB physical memory, with about 60 GiB reported available at this checkpoint; swap is 8 GiB.
- The unencrypted root filesystem is approximately 393 GiB with about 361 GiB currently available. The internal NVMe is approximately 1.9 TiB total, with capacity beyond the current root partition not allocated.
- virt-install and libvirt/QEMU management tools respond normally. No preexisting guest disk images or virtual machines were found. Libvirt default network remains inactive and not configured for autostart.
- Cloud image preparation utilities were not installed at the moment of the package query.
- Proposed test VM allocation: 4 vCPU, 8 GiB RAM and a sparse 40 GiB virtual disk. The future cloud-connected Hermes is intended to run in a KVM virtual machine, not directly as a host Docker container.
- The host root remains unencrypted in this prototype stage. No genuine cloud credentials or private workloads should be put into the VM until secure storage and networking controls are implemented and tested.
- No guest has been created or booted at this checkpoint.

## Cloud image tool installation preflight (owner observation)

- `cloud-localds` and `qemu-img` are present and executable; QEMU image utility reports version 8.2.2; virt-install reports version 4.1.0.
- SSH remains active and host UFW remains enabled with the designated authorized workstation rule.
- The shell transcript includes a malformed/interleaved apt command line followed by an interrupt (^C). No system failure was observed; package manager consistency is **not yet verified**.
- Next action is read-only `dpkg --audit`, installed-package status, and apt/dpkg process inspection. Do not create a guest or run new package installations until the package state is verified.
- Planned first test Cloud Hermes VM remains 2 vCPU, 4 GiB RAM, 40 GiB disk, with no initial network interface and no credentials on the current unencrypted host filesystem.

## Ubuntu Cloud Image downloaded and checksum checked (owner terminal evidence)

- On P2002, the Ubuntu Noble amd64 server cloud QCOW2 image and its SHA256SUMS manifest were downloaded from Ubuntu cloud image distribution.
- sha256sum reported the image `OK` against the downloaded manifest. The manifest signature was not separately verified, so this does not yet independently authenticate its publisher.
- qemu-img recognized QCOW2, 3.5 GiB virtual capacity and about 593 MiB disk usage, with the corrupt flag false.
- No guest virtual disk clone, cloud-init seed, domain definition or VM boot has yet occurred. The planned first guest profile is 2 vCPU, 4 GiB RAM and 40 GiB virtual storage, with networking initially disconnected.
- Before creating guest credentials, independently inspect the administrative ASUS workstation for a suitable existing guest-specific SSH public key. Never create the administrative guest private key on P2002; this protects against recurrence of the earlier wrong-terminal incident.

## Guest-specific SSH key created on administrator workstation (operator evidence)

- Operator confirmed their active terminal was on the designated ASUS administrative workstation rather than P2002, before invoking ssh-keygen.
- A distinct Ed25519 key pair for the planned Cloud Hermes VM was generated on ASUS, with no overwrite of an existing guest key.
- The new public-key fingerprint differs from the previously verified P2002 administrative key. Owner verified private-key mode 0600 and public-key mode 0644. Public fingerprints are deliberately omitted from this public journal to minimize unnecessary operational metadata.
- The guest private key remains on ASUS. The next prescribed step is to copy only the .pub file to the P2002 staging directory and verify its fingerprint there before preparing guest cloud-init.
- At this point public-key transfer, guest creation and guest boot have not yet been confirmed.

## Cloud VM administrative public key successfully staged (operator verification)

- The administrative workstation's separate Cloud Hermes VM public key was copied onto P2002. The operator independently verified that its ED25519 SHA256 fingerprint matches the one generated on the ASUS workstation. The public-key staging file has mode 0644.
- The private key remains on the trusted administrative workstation. The pre-existing P2002 host SSH key remains separate.
- The next planned operation is to copy the checksum-verified Ubuntu cloud image to a dedicated guest QCOW2 under libvirt images and resize the copy to a 40 GiB virtual capacity, leaving the downloaded source unchanged. The guest is not yet created, booted or network-attached.
- An initially networkless guest requires an independent serial-console/recovery plan because SSH cannot work without guest networking.

## Cloud Hermes VM disk created and verified (owner-supplied output)

- Operator copied the checksum-checked Ubuntu 24.04 Cloud Image into a separate QCOW2 under the libvirt image directory, then resized the copy to **40 GiB** virtual capacity.
- qemu-img reports the new file occupies approximately **1.75 GiB** of actual storage at this checkpoint, recognizes QCOW2 and reports corrupt=false. qemu-img check returned **no errors**.
- The original downloaded source image remains a separate 3.5 GiB virtual disk and was not resized.
- Libvirt VM list remains empty: the guest domain has not yet been defined or booted.
- Planned resources: 2 virtual CPUs, 4 GiB RAM and 40 GiB storage, initially with no guest network interface. Cloud-init credentials must include only the separate administrative public SSH key, never the ASUS private key; password SSH and root login are disabled. Without any guest NIC, serial console is for observing boot until a separate permitted management channel is established.

## Cloud-init seed ISO byte-for-byte validation (owner terminal evidence)

- Read-only mounted the generated cidata ISO and inspected its contents. The seed contained the expected user-data and meta-data files.
- Byte-for-byte comparisons (`cmp`) of each file inside the mounted ISO against its staged source returned PASS for both files.
- Operator unmounted the ISO without reported error.
- This validates seed contents prior to VM definition, but not successful cloud-init execution at guest boot. No guest has been booted yet.
- Next controlled stage: check the libvirt staging destination does not exist and ISO is unmounted, copy seed to the libvirt images directory, and compare SHA256 hashes of the copy before VM creation. No secret/private SSH key should be placed in the seed image.

## VM registered and pre-boot inspected (owner-provided output)

- The Cloud Hermes KVM domain was registered persistently but was still powered off at the checkpoint. The registered definition reports two vCPUs, 4194304 KiB (4 GiB) memory, disabled autostart, no network interfaces, a guest QCOW2 disk and a cloud-init ISO.
- A PTY-backed ISA serial console is defined. A virtio guest-agent channel exists, but guest-agent installation and operation have not been verified.
- At the pre-boot inspection, both guest image files were owned by root and had mode 0644, and the image directory had mode 0711. Actual QEMU runtime access and mandatory access controls have not yet been tested. File permissions must be reviewed before any confidential material is stored.
- Host SSH and UFW remained active. No guest network interface was attached.
- Next safe operation is the first offline boot and serial-console observation. No guest boot success, guest cloud-init completion or network connectivity is claimed yet.

## First KVM power-on (owner-verified)

- Operator executed `virsh start` for the previously defined Cloud Hermes domain. Libvirt reported the domain started.
- Subsequent `virsh list --all` showed the guest in `running` state; `virsh domiflist` returned no network interfaces, consistent with the intentionally offline first boot.
- This only establishes hypervisor start and no attached guest NIC. The guest OS boot sequence, serial console, guest cloud-init completion and guest administrative access remain unverified at this checkpoint.
- Next: inspect serial console without changing guest settings. No live cloud credentials have been added.

## Offline guest serial login prompt observed

- The operator attached to the running Cloud Hermes domain using `virsh console` and saw the Ubuntu guest serial login prompt for the expected guest hostname.
- This confirms that guest boot reached a login prompt and that the serial console is operational while no NIC is attached.
- It does not independently establish full cloud-init success, authorized-key enrollment or an administrative login. The account has password login disabled, so no serial password login was attempted.
- Next: detach serial console with Ctrl+] without powering down; confirm domain stays running, then design a narrowly scoped management network that does not bridge the cloud guest onto the private LAN.

## Proposed isolated management network XML validated

- Operator created `cloud-admin-network.xml` for proposed libvirt network `claryel-cloud-admin`, bridge `virbr240`, host-side address 172.30.240.1/24, and DHCP range 172.30.240.10-20. The XML omits a libvirt forwarding element.
- A read-only structural Python check returned five PASS checks and OVERALL PASS. The guest remained running.
- No new network has been started, no VM NIC has been attached, and libvirt network registration is not yet confirmed.
- Security caveat: libvirt isolated-network configuration alone does not prove confinement on a host with IPv4 forwarding enabled by Docker. Verify effective host firewall and east-west access controls before activating/attaching the guest.

## Retrospective verification and correction of missing checkpoints

This checkpoint reconciles the earlier chronological installation notes with newly supplied operator output. Earlier sections are historical snapshots, not current-state assertions.

- Cloud-init user-data was inspected with its public SSH key redacted. The expected guest-specific public key was found, and the installed cloud-init schema command returned `Valid schema user-data`. This validates the input file, not the outcome inside the booted guest.
- The cloud-init seed ISO was copied into the libvirt images directory. The operator's SHA256 output showed the source ISO and its destination copy had identical hashes. A prior attempt to compare before actually copying produced a file-not-found error; this was a command-sequencing mistake, later corrected.
- A dry-run virt-install produced the VM XML. Initial AI-authored validation falsely failed the 4 GiB memory check because it required an explicit `unit="KiB"` attribute. The actual XML omitted `unit`, which means KiB by default, and specified 4194304 KiB; a corrected check returned 4096 MiB PASS. This was an error in the AI validation script, not an actual resource mismatch.
- The KVM domain was defined persistently with 2 vCPU, 4 GiB, a 40 GiB QCOW2 system disk, a cloud-init seed device, serial console, no NIC and disabled autostart. The operator subsequently confirmed first boot to a serial login prompt; VM remained running after leaving the console.
- Guest cloud-init completion, guest user/SSH-key installation, and SSH login have **not** yet been independently verified. The guest has no network interface at this checkpoint.

## Administrative libvirt network registration verified

- Operator executed `virsh net-define` using the previously structurally checked XML. Libvirt reported success.
- `virsh net-list --all` showed `claryel-cloud-admin` as **inactive**, **autostart=no**, **persistent=yes**. The standard libvirt `default` network is also inactive and has autostart disabled.
- `virsh domiflist claryel-cloud-hermes` remained empty: the guest has not been attached to the management network.
- The proposed libvirt network intentionally omits `<forward>`, but **network isolation is not yet accepted**. Host IPv4 forwarding is enabled for Docker, and libvirt/UFW/Docker effective packet-filtering interactions remain to be tested before connecting the VM.
- Current operational decision: do not activate the network, enable its autostart or attach a VM NIC until firewall policy and test methodology are reviewed.

## Open acceptance gaps

1. Verify effective guest initialization and access after a controlled management channel exists; the serial login prompt alone is insufficient.
2. Verify robust isolation against host services, the LAN, the Internet and Docker zones, including both protocol families and actual packet-filtering order; a no-forward libvirt XML is not a security proof.
3. Review confidential-data protection: the host root filesystem is presently unencrypted, and the guest images were observed with mode 0644 before boot. Do not load private production data or cloud credentials yet.
4. Validate the eventual Security Gateway as the exclusive authorized crossing, including application-layer relaying, logging and deny-by-default controls.
5. Outstanding host acceptance: local console through the intended integrated graphics connection is not working; host GPU/Strata workloads and recovery procedure are not yet validated.
6. Verify the cloud image manifest signature separately if independently authenticated supply-chain integrity is required. The SHA256 manifest was downloaded alongside the image, but its signature was not checked.

**Audit basis:** terminal output supplied by operator to ChatGPT, reviewed against the sequential public installation log. No direct shell access to P2002 was used for this retrospective audit.

## nftables readiness for independent Cloud VM guard (owner terminal evidence)

- nftables v1.0.9 is available. The stock nftables systemd service is disabled/inactive; /etc/nftables.conf exists. No existing table named for CLARYEL was reported.
- Existing UFW early INPUT rules include established/related traffic acceptance and a UDP DHCP client response allowance; those host-side rules alone do not determine future guest DHCP behavior.
- Both libvirt networks remain inactive, and the Cloud Hermes VM has no NIC. No guest reachability is claimed.
- Next stage: prepare an independent inet table with input and forward hooks and validate its syntax only. Avoid enabling the stock nftables service or running an unreviewed `flush ruleset`, which could interfere with UFW/Docker/libvirt. Apply rules and arrange persistence only after deliberate review and rollback controls.

## CLARYEL guard nftables rules staged and syntax checked

- Operator created a dedicated nftables configuration file for an `inet claryel_guard` table, with `input` and `forward` hooks at priority -10.
- Proposed guard limits ingress from the future guest bridge to established/related replies plus DHCP client-to-server UDP, dropping other guest-to-host IP input. Forwarding rules drop IP forwarding both into and out of the proposed guest bridge.
- `nft --check --file` printed no syntax errors; full intended file content was displayed in the local terminal. The operator's follow-up `nft list tables` confirmed no CLARYEL table applied yet.
- Both libvirt networks remain inactive. The VM has no guest NIC.
- IMPORTANT: syntax validity is not runtime policy validation or persistence. Before any guest network is activated, apply guard with explicit rollback, verify rules in the live kernel and test ASUS admin SSH remains available. Later arrange independent service persistence without enabling a stock nftables configuration that flushes other stacks' rules.

## CLARYEL guest-bridge guard applied to live nftables (operator evidence)

- Operator checked and loaded the previously staged inet claryel_guard table using nft. The live kernel table was printed afterwards and contains guest_input and guest_forward base chains at hook priority -10.
- guest_input accepts established/related IPv4/IPv6 conntrack responses and DHCP requests from the named future guest bridge, then drops other traffic entering from that bridge. guest_forward drops forwarded traffic entering or leaving that bridge.
- Host ssh systemd service remained active after the change, but an independent new SSH connection from the administrative workstation has not yet been observed for this checkpoint.
- Both libvirt networks remain inactive and the Cloud Hermes VM still has no NIC. The firewall rules' actual packet-level effectiveness against guest traffic and persistence across reboot remain UNTESTED.
- Next acceptance gate: verify a new SSH login from the ASUS admin workstation while retaining the existing connection, then establish narrowly scoped persistence and negative/positive network tests before attaching a VM NIC.

## Fresh SSH check after guard activation

An independent new SSH session from the ASUS administrator workstation reached P2002 and confirmed the SSH service remained active. Persistence of the guard and guest traffic behavior are not yet tested.

## Stock nftables systemd unit review

The owner inspected the installed nftables systemd unit and configuration. The stock service is disabled. Its service stop flushes the entire nftables ruleset, and the configured startup file includes a full ruleset flush. Therefore, enabling that stock service without redesign risks disrupting UFW, Docker, libvirt, and CLARYEL rules. Docker and libvirtd services are enabled; SSH and the previously installed CLARYEL guard remain active. Next step is to draft a separate, non-flushing CLARYEL-only systemd service and loader for inspection before enabling or rebooting. No network or service change was made in this review.

## Systemd guard unit drafted and verified locally

The operator staged a dedicated oneshot systemd unit with a non-flushing loader path, start ordering before Docker and libvirtd, and no ExecStop command. `systemd-analyze verify` reported only that the loader executable does not yet exist; this is expected at the draft stage. The live guest-bridge guard table remained present and SSH stayed active. No unit or loader has been installed/enabled, no guest interface attached and no network activated. Next, stage and syntax-check an idempotent guard loader without applying it.

## Guard loader staging syntax check

Owner staged the CLARYEL guard Bash loader script in the VM preparation directory and inspected its full content. `bash -n` passed; the live CLARYEL firewall table remained present and host SSH remained active. The loader is designed to validate its rules file, return early if the guard table already exists, otherwise load just that rules file and confirm the table exists. No system installation or invocation of the loader has yet been reported. Known gap: existence-only guard detection could mistake a partially configured or stale table for a complete policy; content validation and safe recovery need separate acceptance before production deployment. Next: install staging copies with non-overwrite checks, then inspect systemd verification without starting/enabling the unit.
