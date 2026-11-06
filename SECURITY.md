# Security and Responsible Disclosure

This repository is public. All contributions, including direct GitHub API writes by AI assistants, must comply with [the publication security policy](policies/PUBLICATION-SECURITY.md). An approved human controls disclosure.

This policy draws on the existing security practice in the adjacent [claryel-autonomous](https://github.com/ViktorIXAI/claryel-autonomous/blob/main/SECURITY.md) project: Gitleaks, infrastructure-pattern checks, local-only configuration, scanning publication artifacts, protected history and credential-incident procedures. It strengthens that practice with a **separate full-fidelity private recovery ledger**, deterministic sanitized public reproduction steps, and an explicit mandatory **pre-publication** gate for API-based authors.

## Report a vulnerability privately
Use the repository's GitHub private vulnerability reporting or security advisory facility when enabled, or contact the maintainer through an already-established private channel. **Never submit a usable credential, sensitive IP/host inventory, customer data or exploit detail as a public issue.**

## Suspected disclosure
1. Stop every publishing automation and cancel public uploads.
2. Identify affected commits, branches, issue text, assets, pull requests, caches and forks in a **private** incident record.
3. Revoke or rotate exposed credentials first. Assume publicly exposed secrets are compromised, whether or not misuse is visible.
4. With owner approval, sanitize public content and evaluate necessary history/release removal, cached replicas and notification/legal duties. Preservation of history does not override secrecy or GDPR.
5. Publish only a redacted incident note and preventive actions; test restored publishing gates.
6. Retain detailed investigation records only in the separately secured private evidence store.

**Current implementation state:** written policy and example scanner configuration only. Do not claim enforced branch protection, pre-push hooks, GitHub Actions, automated redaction, or a deployed private ledger until independently verified.
