# Mandatory Contributor and Agent Provenance Policy

**Effective 2026-11-06; owner-directed.** Applies to **every new public write** to `ViktorIXAI/claryel`, regardless of the GitHub login used. This is a documentary requirement **not yet technically enforced** by an authenticated publishing gate; see [issue #6](https://github.com/ViktorIXAI/claryel/issues/6) and [issue #1](https://github.com/ViktorIXAI/claryel/issues/1).

## Why this is necessary

Different ChatGPT conversations, model providers, human operators, Hermes agents, automated scripts and other assistants can all use the same GitHub account. The GitHub author/committer account records the credential that published a change, **not a verifiable individual writer or executor**. Never attribute an action to the repository owner merely because the GitHub account is theirs.

## Required declarations for every new contribution

- **Prepared-by:** entity that authored the contribution: human, model/agent or process. Name the provider/product/model **only if actually known**; otherwise use `UNKNOWN`. Different chat instances must have separate safe *declared* writer aliases, for example `CHATGPT-REVIEW-01` versus `CHATGPT-INSTALL-01`, without exposing private chat/session URLs or IDs.
- **Executed-by:** entity that ran the command on a real machine, or `NOT_APPLICABLE` / `UNKNOWN`. A remote assistant receiving user-pasted output is **not** the executor.
- **Evidence-source:** `OWNER_REPORTED`, `TOOL_OBSERVED`, `SIMULATED`, `INDEPENDENTLY_VERIFIED`, `PROPOSED`, or `UNKNOWN`; indicate scope and any limitations.
- **Reviewed-by:** entity that actually reviewed the content, or `NOT_REVIEWED`. Never claim independent review if a single assistant reviewed its own work.
- **Approved-by:** owner authorization reference only when evidenced, otherwise `NOT_CONFIRMED`; a general instruction to write does not authorize destructive host changes.
- **Published-by:** credential/account or agent that sent the GitHub write; distinguish the credential from the drafting agent.
- **Attribution-confidence:** `DECLARED_UNVERIFIED` by default, or `VERIFIED` only with a documented independent mechanism; avoid treating self-reported identity as authenticated.
- **Related-checkpoint:** stable public ID or `NOT_APPLICABLE`.

For **Git commits**, add the following trailers to each commit message where the publishing tool allows it:

```text
Prepared-by: ChatGPT (declared alias CHATGPT-REVIEW-01; model UNKNOWN)
Executed-by: NOT_APPLICABLE
Evidence-source: TOOL_OBSERVED
Reviewed-by: SELF_REVIEW_ONLY
Approved-by: OWNER_REQUEST_2026-11-06
Published-by: GitHub API via shared ViktorIXAI account
Attribution-confidence: DECLARED_UNVERIFIED
Related-checkpoint: NOT_APPLICABLE
```

Do not copy the example literally unless it is true for that specific operation.

For **issues, PRs, comments, reports, logs and releases**, include a compact visible **Contribution provenance** block with the same fields. Direct API writers must attribute their writes even if the commit message API lacks custom identity support. Existing history must **not** be rewritten to fabricate attribution; use forward-looking compliance, and independently verified corrections only.

## Differentiating authors and actions

One contribution can involve several entities: an AI authors a proposed command; the owner runs it; a second AI interprets sanitized output; a GitHub connector publishes the summary. Record each role distinctly. A `PASS` printed by a script is a test observation, not a security certification. Never infer specific model version, conversation, agent identity or host executor without evidence.

## Security and enforcement

- Writer aliases are public **pseudonyms**, not credentials, unique authenticated identifiers, or user tracking identifiers.
- Never publish conversation URLs, access tokens, machine identifiers, private account names or personal data to disambiguate AI instances.
- The future pre-publication gate must validate required fields on every outgoing surface and compare actor claims against available private attestations. Missing/ambiguous information must fail closed where mandatory and be stated as `UNKNOWN` rather than invented.
- The same writer must not be able to self-approve security exceptions; policy changes require owner review.
- Until a gate is implemented and independently tested, **compliance is voluntary and cannot be proven solely by this file**.

## Provenance of this policy publication

- Prepared-by: ChatGPT (this repository-audit conversation; model GPT-6, declared)
- Executed-by: NOT_APPLICABLE (documentation only)
- Evidence-source: TOOL_OBSERVED (GitHub repository files and commits)
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner request in this conversation (publication of attribution requirement)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: NOT_APPLICABLE
