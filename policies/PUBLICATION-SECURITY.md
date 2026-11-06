# CLARYEL Publication Security: Two-Ledger Reproducibility Policy

**Status: mandatory publication policy, effective 2026-11-06. Implementation and CI enforcement require separate testing and owner approval.**

> **Temporary fast-log window (owner request):** From 2026-11-06 16:06 UTC through 2026-11-07 16:06 UTC, consult [the temporary 24-hour publication exception](TEMPORARY-FAST-LOG-WINDOW-2026-11-06.md). Non-sensitive technical logs may be published verbatim without editorial delay; secret/credential/third-party-privacy and material infrastructure exposure checks remain mandatory. The exception expires automatically, and this standing policy then applies unchanged.

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
- Replace environment-specific values with descriptive placeholders (e.g., `ADMIN_CLIENT_IPV4`, `BOX_LAN_IPV4`, `LAN_IFACE`, `RECOVERY_DEVICE_BY_ID`, `SSH_USER`, `SECRET_REF`). Do not publish observed addresses or confidential host naming.
- Show *how to discover* and *validate* each local parameter without publishing its value. Distinguish owner-observed, agent-proposed, independently tested and not tested. Public runbooks must be implementable on different hardware and networks.
- Publish linkable public checkpoint IDs and non-reversible **sanitized artifact** hashes only. Do not hash low-entropy secrets and publish their hashes; use salted keyed attestations or private references as needed.
- Do not silently paraphrase an exact local procedure when differences affect recovery. Record a private-to-public mapping of substitutions and verified command equivalence in the private ledger.

## Forbidden public content (including unintentional metadata)

Secrets: API tokens, passwords, authentication cookies, SSH private keys, LUKS recovery material, credential-containing URLs, .env values, vault dumps, encryption passphrases, TOTP seeds, cert private keys, service account JSON, cloud session/browser profiles.

Infrastructure: **all actual private IPs (RFC1918), IPv4 link-local, loopback if used in real logs, global/ULA/link-local IPv6 host addresses, MACs, real SSH destinations/port mappings, real interface-to-host mappings, DNS hostnames/domains identifying the owner's network, router configuration details, VPN/Tailscale/mesh addresses, serial numbers, storage UUIDs, machine-id/boot-id and unique device IDs.** Reserved documentation ranges and synthetic values are allowed only when explicitly identified as examples.

Personal/customer data: names/usernames, paths embedding local accounts, real home directories, invoices, personal communications, health records, user content, unredacted telemetry or screenshots containing identifying data. Avoid publishing account-specific or customer-specific hashes, filenames, and exact timestamps when linkable to identities.

Untrusted/raw materials: core dumps, crash traces, database backups, container/VM exports, shell history, tool transcripts, provider responses, logs and artifacts **must be treated as private by default**. Their filenames or exclusion rules do not constitute authorization to publish them.

## Mandatory publication gate

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

- Flag RFC1918 IPv4, IPv4 link-local, non-example IPs in terminal dumps, IPv6 addresses other than standard documentation prefixes when they appear to reveal host/network information, MAC/EUI-64 patterns, SSH targets, absolute per-user paths, machine-id/UUIDs, hostname and URL identities, credentials and service tokens.
- Flag base64, JWT, private key blocks, high-entropy strings and secrets in binary metadata/EXIF; review false positives carefully without leaking captured value into scanner logs.
- Disallow direct public publication of Linux `ip addr`, `ip route`, `hostnamectl`, `ssh -v`, `journalctl`, `virsh dumpxml`, Docker inspect, Kubernetes YAML, env dumps, `lsblk -f`, `cryptsetup luksDump` and analogous output unless rendered as reviewed sanitized summaries.
- No default exceptions for synthetic data containing plausible live credentials or IPs that could be mistaken for real values. Use documentation-only IP blocks (IPv4 192.0.2.0/24, 198.51.100.0/24, 203.0.113.0/24, IPv6 2001:db8::/32) when worked examples require numeric addresses, and label them synthetic.
- Never publish master key recovery material, local publication tool configuration or an exact unlock procedure that could bypass authorization. Reproducible public instructions must include creation/rotation steps for each installer's **own** keys.

## Operational privileges and retention

- Human owner approves changes to disclosure gates and access policy. AI agents are untrusted authors, never authorized to bypass the gate, self-approve exceptions or publish from a private vault.
- Use isolated GitHub write identities with least privilege; protected main, no force-push by default, no deletion rights except explicitly authorized incident remediation. Enforce once possible; current configuration is unverified.
- Keep append-only correction history of non-secret mistakes. If a secret leaks, prioritize revocation and controlled removal; a public correction must not reproduce it.
- Classify file retention, customer rights and backup expiration in accordance with applicable obligations. Public Git is not an encrypted or reversible private backup.
- Every installation step: `private-checkpoint ID -> sanitized public step -> test evidence -> next owner-approved operation`. No claims that a not-yet-built publishing gate or backup is already protecting data.

## Test criteria (future)

- Insert synthetic canary tokens, RFC1918 IP, IPv6 ULA, MAC, user home paths, session cookie, SSH private key marker and credentials into staged test fixtures; verify the gate blocks them **before** upload.
- Confirm the public counterpart remains sufficient to reproduce firewall, KVM, Docker, recovery USB and LUKS operations using substituted test values.
- Confirm private records can rebuild exact P2002 procedure offline, while public artifacts reveal no real infrastructure facts.
- Exercise direct GitHub API paths, issue comments, releases and binary files; prove bypasses fail closed. Repeat after pipeline changes.

**Limitations:** Gitleaks plus regexes can miss unknown secret formats and semantic/contextual disclosures. Human review and independent checks remain required; secrecy is not proven merely because scanners pass.
