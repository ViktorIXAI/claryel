# Temporary 24-hour owner-authorized P2002 technical-log publication window

**Owner authorization:** explicit instruction in the active ChatGPT conversation on 2026-11-06.  
**Starts:** 2026-11-06T17:09:44+01:00 (16:09:44 UTC).  
**Expires automatically:** 2026-11-07T17:09:44+01:00 (16:09:44 UTC).  
**Scope:** P2002 / CLARYEL installation, configuration, debugging, security review, and agent activity during this window.

## Publication authorization

The owner explicitly authorizes comprehensive, prompt publication of technical activity and evidence for the scope above. During this window, publish technical logs and diagnostic outputs without suppressing non-secret operational detail, including actual host/network identifiers, private IP addresses, MAC addresses, hostnames, VM identifiers, topology, interface mappings, firewall/access rules, service configuration, command output, failures, warnings, and agent mistakes.

This is a time-limited owner override of contrary non-secret infrastructure-disclosure restrictions in the standing publication and installation-logging policies and the logging restrictions referenced by issues #1 and #4. It applies only to public technical reporting for the stated P2002/CLARYEL work. It does not amend the permanent acceptance criteria for the publishing gate or reproducibility tests.

## The only excluded material

Never publish:
- passwords, passphrases, API tokens, access tokens, authentication cookies, session credentials;
- private keys, recovery phrases/codes, TOTP seeds, certificate private keys, vault contents, or other secret authentication material.

If any such material appears in an otherwise publishable log, withhold or redact the secret value before upload. Do not include scanner output that reproduces a detected secret. No secret or password is authorized for publication.

## Required handling

- Keep Gitleaks and credential detection enabled; do not disable scanners or remove existing rules.
- Infrastructure-only findings may be published under this explicit owner authorization during the window; credential/secret findings remain hard blocks.
- Review the exact outgoing bytes before each write. This is a manual review process, not evidence that the future fail-closed publication gate has been implemented.
- Preserve truthful provenance. Distinguish owner-supplied terminal output from commands personally executed by an agent. Do not fabricate event timestamps, exit codes, machine identities, or independent verification.
- Use `UNKNOWN` or `NOT_TESTED` where evidence is absent.
- Keep all changes to the P2002 host, VM, network, firewall, and disks under their separate execution approvals. This publication authorization is not permission to execute host changes.
- Do not delete evidence, rewrite Git history, publish private repositories, or claim that publication makes public data private again.

## Expiration

At **2026-11-07T17:09:44+01:00**, this exception expires automatically without further approval. The standing publication-security, installation-audit-logging, and contributor-provenance policies resume without this temporary override. Check the absolute expiry before every publication.

## Contribution provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: NOT_APPLICABLE
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: NOT_APPLICABLE (policy documentation only)
- Evidence-source: OWNER_REQUEST
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: explicit owner instruction, 2026-11-06
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
