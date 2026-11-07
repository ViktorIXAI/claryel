# Security and Responsible Disclosure

This repository is public. All contributions, including direct GitHub API writes by AI assistants, must comply with [the publication security policy](policies/PUBLICATION-SECURITY.md). The owner has issued a standing authorization for prompt publication of routine non-secret CLARYEL engineering logs and project-owned lab operational/network details; no automated publisher-readiness hold applies. Credentials, private keys, customer/third-party confidential information and unrelated personal data remain prohibited.

This policy draws on the existing security practice in the adjacent [claryel-autonomous](https://github.com/ViktorIXAI/claryel-autonomous/blob/main/SECURITY.md) project: Gitleaks, infrastructure-pattern checks, local-only configuration, scanning publication artifacts, protected history and credential-incident procedures. It strengthens that practice with a **separate full-fidelity private recovery ledger**, deterministic sanitized public reproduction steps, and an explicit mandatory **pre-publication** gate for API-based authors.

## Report a vulnerability privately
Use the repository's GitHub private vulnerability reporting or security advisory facility when enabled, or contact the maintainer through an already-established private channel. **Never submit a usable credential, customer data, third-party infrastructure details or secret exploit material as a public issue.** Non-secret P2002 laboratory IPs, container/network names and architecture details are intentionally public under the owner's [standing disclosure policy](policies/PUBLIC-ARCHITECTURE-AND-CLIENT-IDENTITY.md); do not extend that permission to customer or unrelated networks.

## Suspected disclosure
1. Stop every publishing automation and cancel public uploads.
2. Identify affected commits, branches, issue text, assets, pull requests, caches and forks in a **private** incident record.
3. Revoke or rotate exposed credentials first. Assume publicly exposed secrets are compromised, whether or not misuse is visible.
4. With owner approval, sanitize public content and evaluate necessary history/release removal, cached replicas and notification/legal duties. Preservation of history does not override secrecy or GDPR.
5. Publish only a redacted incident note and preventive actions; test restored publishing gates.
6. Retain detailed investigation records only in the separately secured private evidence store.

**Current implementation state:** written policy and example scanner configuration only. Do not claim enforced branch protection, pre-push hooks, GitHub Actions, automated redaction, or a deployed private ledger until independently verified.
