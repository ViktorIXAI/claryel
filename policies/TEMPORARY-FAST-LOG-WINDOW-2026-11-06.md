# Temporary 24-hour fast-log publication window

**Owner request:** 2026-11-06. **Starts:** 2026-11-06 17:06 Europe/Rome (16:06 UTC). **Expires automatically as a policy exception:** 2026-11-07 17:06 Europe/Rome (16:06 UTC). **Status:** temporary documentary operating mode, not a change to GitHub permissions or proof of technical enforcement.

## Limited relaxation for this window

The owner requests rapid and comprehensive publication of installation/debugging logs while multiple agents work concurrently. For this 24-hour period, **technical logs that contain no secrets, no other people's private data, and no exploitable environment-specific access details may be published verbatim**, including failed commands, stack traces, test outputs, warnings, model/agent mistakes, configuration reasoning, time stamps and non-sensitive hardware/software versions. There is **no requirement to rewrite such safe text into a prose narrative**, create separate rich evidence packets, or await editorial style/completeness review. A clear `UNVERIFIED` label is sufficient for incomplete results; missing extra metadata should be corrected later rather than used to suppress an otherwise safe technical log.

Publication remains English-first as the project's norm, but language formatting alone must not stall a safe urgent technical checkpoint. Maintain clear per-instance agent attribution and machine aliases if available; do not fake missing identities. For Hermes, runtime machine ID remains mandatory: resolve to a safe machine alias before public publication.

## Safety protections NOT relaxed

**Never publish** secrets (API tokens, passwords, private keys, session cookies, recovery phrases), customer or third-party personal content, sensitive files or backups, or live details that make existing infrastructure materially easier to compromise (private IP/host maps, real SSH destinations, credential-bearing command lines, sensitive firewall exceptions, account-specific dumps). Even a 24-hour public exposure can be copied indefinitely. If a raw artifact includes any such data, **hold the raw artifact outside public GitHub** and publish only a redacted copy or a concise safe summary. Do not disable Gitleaks, credential filters, or other existing detection checks. Do not bypass the single-publisher security gate once deployed.

This exception does not grant permission to execute changes on the machines, delete evidence, rewrite Git history, alter isolation/security controls or publish private repositories. It does not attest that current prepublication gate enforcement exists.

## Automatic expiration of policy exception

At **2026-11-07 16:06 UTC**, this exception becomes void **without further action or approval**. The standing [publication security policy](PUBLICATION-SECURITY.md), [installation audit logging policy](INSTALLATION-AUDIT-LOGGING.md), and [contributor provenance policy](CONTRIBUTOR-PROVENANCE.md) then apply without relaxation. Automated agents MUST evaluate the absolute expiry time before every publication and not rely on the presence of this file as a permanent waiver. Technical CI enforcement has not been validated.

## Contribution provenance

- Writer-instance: CHATGPT-REVIEW-01
- Writer-role: REPOSITORY_AUDITOR
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: NOT_APPLICABLE
- Evidence-source: OWNER_REQUEST
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-06
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
