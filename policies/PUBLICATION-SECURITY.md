# CLARYEL Publication Security: Two-Ledger Reproducibility Policy

**Status: mandatory publication policy, effective 2026-11-06. Implementation and CI enforcement require separate testing and owner approval.**


## Standing owner authorization: prompt publication of technical logs (2026-11-07)

**This is a permanent standing instruction until the owner gives a later explicit instruction changing or revoking it. It is not a 24-hour exception.**

The owner directs CLARYEL contributors and agents to publish routine project engineering records promptly and comprehensively in this repository. **Do not delay publication because an automated publisher, pre-publication gate, CI workflow, scanner integration, or private evidence ledger is unfinished or unavailable.** Manual preparation, review and GitHub publication are authorized. The old rule to wait for the publisher/gate to become ready is superseded for the scope below.

The authorized public record includes commands, terminal output, diagnostics, failed attempts, DNS/routing/network findings, errors, warnings, mistaken assumptions, corrections, configuration changes, service state, versions, tests, results, residual risks and next actions. For the owner-controlled P2002/CLARYEL reference lab—including its Cloud Hermes guest—non-secret internal network and operational details are publishable: private lab IP addresses, host/VM/service/network/bridge names, interfaces, routes, ports, firewall and access rules, package and software versions, and relevant systemd/libvirt/nftables output. These details are not to be withheld merely because they describe the internal lab network.

For ordinary text-based engineering logs, the required pre-publication check is a review of the exact outgoing text to remove the prohibited material below and to preserve accurate provenance. Run automated scanners when available, keep them enabled, and record their results; however, scanner or publisher readiness is not a prerequisite for routine logs. A scanner failure must never be represented as a pass. If a possible credential or protected personal/customer/third-party datum cannot be confidently excluded, redact or withhold that material—not the unrelated technical log.

This standing authorization does **not** permit publication of passwords, passphrases, API/OAuth/access tokens, cookies, session credentials, private keys, recovery material, secret environment-variable values, credential-bearing URLs, customer/third-party confidential data, or unrelated personal information. Do not publish raw secret-bearing files, vault contents, private keys, backups, or unreviewed binary/archive artifacts. Customer and third-party deployments remain subject to generated placeholders and the per-client identity rules; this authorization is for CLARYEL-owned laboratory activity.

Preserve chronology and mistakes. Never invent commands, outputs, timestamps, actors, exit codes, approvals or verification. Label owner-supplied terminal transcripts as owner-supplied; distinguish proposed, executed, observed and independently verified work. Append corrections rather than silently rewriting history.

This owner instruction supersedes conflicting temporary publication holds and any requirement to wait for the automated publisher for the scope above. Only a later explicit owner instruction may change this standing rule. It does not weaken the secret, customer-data or personal-data exclusions, and it does not authorize host/network changes.


> **Standing owner direction — 2026-11-07:** The owner directs public disclosure of the P2002 laboratory architecture for independent security review, including its actual non-secret internal IP addresses, host/container/VM/service names, Docker network names, ports, routes, firewall/access configuration, versions, commands, outputs, failures and test results. This standing direction supersedes the expired 24-hour exception for the P2002 lab's non-secret architecture and operational identifiers. It does not authorize credentials, private keys, customer data or unrelated personal/network identifiers. See [the public architecture and client identity standard](PUBLIC-ARCHITECTURE-AND-CLIENT-IDENTITY.md). This does not prove technical gate enforcement or deploy the private evidence ledger.

## Scope and origin

Applies to **every** public GitHub surface: commit contents and history, branches, tags, diffs, commit messages, issues, PRs, comments, reviews, discussions, release assets, Actions logs/artifacts, dashboards, screenshots, terminal output, binary attachments, model artifacts and AI-agent direct API writes. Publication is **English only**.

Adapted from the adjacent claryel-autonomous repository's SECURITY.md, .gitleaks.toml and .gitignore (read 2026-11-06). Preserve its built-in Gitleaks plus infrastructure detectors, staging/archive and history scans, local publish.env, blocking credential findings, non-destructive history preference and private breach remediation. The CLARYEL extensions below take precedence where stricter.

## Two independent evidence ledgers

**PRIVATE FULL-FIDELITY LEDGER (not in this public repository):**
- Operator-controlled, offline-capable, encrypted, access-restricted local evidence storage with separate recovery credentials and tested restore capability.
- Preserve exact execution host, source and target IP/IPv6, MAC/hostname where truly needed, adapter identity, precise command lines, network topology, key *references* (not raw key bytes in routine logs), disk IDs, software versions, expected/actual outputs, error codes, approvals, timestamps, and procedure/checkpoint IDs.
- Protect raw screenshots/logs, config exports and secrets separately with least privilege and retention rules; do not assume current test root disk is encrypted. While that disk is not encrypted, use only non-sensitive synthetic information and defer storage of real full-fidelity secrets/infrastructure history to a suitably protected destination.
- Never copy the master secret vault into logs. Credentials must be stored only in a separately encrypted authorized vault; references can point to vault identifiers.
- Maintain integrity evidence (hashes, signatures when deployed), access records, secure offline copies and eventual backup/restore testing. **This private repository/storage location has not yet been deployed**.

**PUBLIC REPRODUCIBILITY LEDGER (this repository):**
- Publish hardware *classes*, operating-system/application versions, non-identifying architecture, exact deterministic commands with variables, synthetic fixtures, package pin/digest references, tests, expected results, sanitized results, failures, reasoning and timestamps.
- For the P2002 reference lab, publish the actual non-secret lab identifiers and network values covered by the owner's standing direction. For every client/third-party deployment, replace environment-specific values with descriptive placeholders (e.g., `ADMIN_CLIENT_IPV4`, `BOX_LAN_IPV4`, `LAN_IFACE`, `SSH_USER`, `SECRET_REF`) and do not publish that customer's live addresses, hostnames or deployment manifest. See [the per-client identity standard](PUBLIC-ARCHITECTURE-AND-CLIENT-IDENTITY.md).
- Show *how to discover* and *validate* each local parameter without publishing its value. Distinguish owner-observed, agent-proposed, independently tested and not tested. Public runbooks must be implementable on different hardware and networks.
- Publish linkable public checkpoint IDs and non-reversible **sanitized artifact** hashes only. Do not hash low-entropy secrets and publish their hashes; use salted keyed attestations or private references as needed.
- Do not silently paraphrase an exact local procedure when differences affect recovery. Record a private-to-public mapping of substitutions and verified command equivalence in the private ledger.

## Forbidden public content (including unintentional metadata)

Secrets: API tokens, passwords, authentication cookies, SSH private keys, LUKS recovery material, credential-containing URLs, .env values, vault dumps, encryption passphrases, TOTP seeds, cert private keys, service account JSON, cloud session/browser profiles.

Infrastructure: Customer, third-party and unrelated production infrastructure identifiers remain forbidden by default, including their actual private/public IPs, IPv6 addresses, MACs, SSH destinations, interface-to-host mappings, DNS names, router/VPN/mesh addresses, serial numbers, storage UUIDs, machine IDs and unique device IDs. **Exception:** actual non-secret P2002 laboratory IP addresses, container/VM/service names, Docker network names, ports, routes and firewall/access topology are explicitly authorized for public disclosure by the owner under [the P2002 public architecture policy](PUBLIC-ARCHITECTURE-AND-CLIENT-IDENTITY.md). Do not extend that exception to customer installations or other networks without a separate owner decision. Public-WAN addresses and personal/customer location identifiers are not included in this exception.

Personal/customer data: names/usernames, paths embedding local accounts, real home directories, invoices, personal communications, health records, user content, unredacted telemetry or screenshots containing identifying data. Avoid publishing account-specific or customer-specific hashes, filenames, and exact timestamps when linkable to identities.

Untrusted/raw materials: core dumps, crash traces, database backups, container/VM exports, shell history, provider responses, and unreviewed logs/artifacts remain private by default. P2002 command transcripts and diagnostic excerpts may be published only after review for excluded secrets, personal/customer data and unrelated identifiers under the owner's standing lab-disclosure direction. Filenames or exclusion rules do not constitute authorization.

## Mandatory publication gate for material outside the standing text-log authorization

The full gate below remains mandatory for automated/bulk publication, releases, binary or archive artifacts, raw exports, and material outside the standing owner authorization above. For routine text-based CLARYEL engineering logs within that authorization, use the manual exact-content review described above and publish promptly; automated publisher, scanner integration, CI enforcement and private-ledger readiness are not prerequisites.


1. **Stage privately**, never by writing the raw artifact directly to the public branch or GitHub API.
2. Classify inputs: public/synthetic, confidential environment, secret/credential, personal/customer or unknown. Unknown is private until reviewed.
3. Create the sanitized public artifact separately; use placeholders with a documented local substitution map. Keep raw local records out of the public Git tree.
4. Scan the **exact bytes to be uploaded** (including issue/comments/messages and screenshots/assets) with built-in Gitleaks plus custom infrastructure detectors, PII heuristics and filename/archive/binary metadata checks; run language and schema/content validation.
5. Scan **git history and branches/tags**, plus release archives and Actions artifacts, for independent leakage. Old history must not be silently assumed clean.
6. Review the complete diff and metadata for semantic leakage (including indirect network topology, cross-file reconstruction, and traces not caught by regex); an independent human approval is mandatory for secrets, permissions or policy changes.
7. Fail closed on a detection, scan error, unknown binary, missing artifact manifest or unresolved ambiguity. Whitelisting requires a reviewed per-finding rationale; blanket global `gitleaks:allow` or disabling default Gitleaks detectors is prohibited.
8. Publish a *specific checked immutable artifact* via least-privilege, single-repo credentials after a successful gate. Recheck that the published file/commit bytes match the approved sanitized version.
9. Record a redacted public audit entry: gate revision, timestamp, actor/provenance, artifact digest, tests and limitations; private ledger stores detailed evidence.
10. Repeat on every update, agent action and release. GitHub Actions post-commit scans are **detection only**, not a substitute for a pre-publication gate. A direct connector/API write must pass the same gate.

## High-value safety checks

- Continue to flag RFC1918 IPv4, IPv4 link-local, non-example IPs in terminal dumps, IPv6 addresses other than standard documentation prefixes, MAC/EUI-64 patterns, SSH targets, absolute per-user paths, machine-id/UUIDs, hostname and URL identities, credentials and service tokens. A finding in a P2002 lab artifact requires explicit scope review against the owner authorization; do not create a blanket scanner bypass or suppress secret/PII findings.
- Flag base64, JWT, private key blocks, high-entropy strings and secrets in binary metadata/EXIF; review false positives carefully without leaking captured value into scanner logs.
- Disallow direct public publication of Linux `ip addr`, `ip route`, `hostnamectl`, `ssh -v`, `journalctl`, `virsh dumpxml`, Docker inspect, Kubernetes YAML, env dumps, `lsblk -f`, `cryptsetup luksDump` and analogous output unless rendered as reviewed sanitized summaries.
- No default exceptions for synthetic data containing plausible live credentials or IPs that could be mistaken for real values. Use documentation-only IP blocks (IPv4 192.0.2.0/24, 198.51.100.0/24, 203.0.113.0/24, IPv6 2001:db8::/32) when worked examples require numeric addresses, and label them synthetic.
- Never publish master key recovery material, local publication tool configuration or an exact unlock procedure that could bypass authorization. Reproducible public instructions must include creation/rotation steps for each installer's **own** keys.

## Operational privileges and retention

- The human owner controls disclosure policy. The standing owner authorization above is approval for routine text-based CLARYEL engineering-log publication; AI agents may prepare and publish those logs after the required manual secret/privacy review. Agents may not widen the authorized scope, self-approve new exceptions, publish from a private vault, or bypass the hard exclusions. The full gate remains mandatory for material outside the standing authorization.
- Use isolated GitHub write identities with least privilege; protected main, no force-push by default, no deletion rights except explicitly authorized incident remediation. Enforce once possible; current configuration is unverified.
- Keep append-only correction history of non-secret mistakes. If a secret leaks, prioritize revocation and controlled removal; a public correction must not reproduce it.
- Classify file retention, customer rights and backup expiration in accordance with applicable obligations. Public Git is not an encrypted or reversible private backup.
- Every installation step: `private-checkpoint ID -> sanitized public step -> test evidence -> next owner-approved operation`. No claims that a not-yet-built publishing gate or backup is already protecting data.

## Test criteria (future)

- Insert synthetic canary tokens, RFC1918 IP, IPv6 ULA, MAC, user home paths, session cookie, SSH private key marker and credentials into staged test fixtures; verify the gate blocks them **before** upload.
- Confirm the public counterpart remains sufficient to reproduce firewall, KVM, Docker, recovery USB and LUKS operations using substituted test values.
- Confirm public P2002 artifacts intentionally expose the actual non-secret lab architecture and identifiers needed for independent review, while customer deployments use generated unique identities and do not disclose their live infrastructure values.
- Exercise direct GitHub API paths, issue comments, releases and binary files; prove bypasses fail closed. Repeat after pipeline changes.

**Limitations:** Gitleaks plus regexes can miss unknown secret formats and semantic/contextual disclosures. Human review and independent checks remain required; secrecy is not proven merely because scanners pass.
