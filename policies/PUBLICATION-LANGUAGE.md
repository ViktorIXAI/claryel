# Mandatory public language and disclosure policy

All public-facing project materials must be in **English only**: READMEs, architecture documents, requirements, source-code comments, GitHub issues, pull requests, reviews, discussions, commit messages, changelogs, release notes, test evidence, logs and agent-generated reports.

Translate non-English instructions before publication, without exposing original private conversations. Product names, standard protocol identifiers, code syntax and legal titles are exempt from translation where necessary. This policy applies to every agent and human contributor; agent-produced content requires review before publication.

Do not publish secrets, credentials, tokens, real customer data, identifying network diagrams, private IPs/hostnames, local filesystem paths or unsanitized telemetry. Run secret and personal-data checks plus human diff review before publishing. Do not claim tests, compliance or operational protection that has not been demonstrated. Exceptions require explicit owner approval; never let an AI agent waive publication rules.

## Mandatory two-ledger publication security

Apply [the Publication Security policy](PUBLICATION-SECURITY.md) to all GitHub entry points including direct API/connector writes. The exact local runbook belongs in an independently encrypted offline-capable private ledger. Public reports contain sanitized parameterized commands and validation evidence, never live network addresses, credentials or identifying hardware material. `.gitignore` alone does not block GitHub API uploads; every public payload requires pre-publication checking. No CI or scanner gate may be claimed deployed without separate proof.
