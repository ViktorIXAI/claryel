# Mandatory Contributor and Agent Provenance Policy

**Effective 2026-11-06; owner-directed.** Applies to **every new public write** to `ViktorIXAI/claryel`, regardless of the GitHub login used. This is a documentary requirement **not yet technically enforced** by an authenticated publishing gate; see [issue #6](https://github.com/ViktorIXAI/claryel/issues/6) and [issue #1](https://github.com/ViktorIXAI/claryel/issues/1).

## Why this is necessary

Different ChatGPT conversations, model providers, human operators, Hermes agents, automated scripts and other assistants can all use the same GitHub account. The GitHub author/committer account records the credential that published a change, **not a verifiable individual writer or executor**. Never attribute an action to the repository owner merely because the GitHub account is theirs.

## Multiple Hermes instances: exact declared instance required

**The generic label `Hermes` is not sufficient.** Every deployed Hermes instance (cloud/public, private/internal, staging, recovery, or other) must have a distinct persistent **public-safe instance alias** plus its role and deployment/box alias. Examples are illustrative, not claims of deployment:

| Writer instance alias | Security role | Deployment alias |
| --- | --- | --- |
| `HERMES-PUBLIC-01` | `PUBLIC_CLOUD_AGENT` | `BOX-LAB-A` |
| `HERMES-PRIVATE-01` | `PRIVATE_LOCAL_AGENT` | `BOX-LAB-A` |
| `HERMES-PRIVATE-02` | `PRIVATE_LOCAL_AGENT` | `BOX-LAB-B` |

Different Hermes instances on the **same host** still require distinct instance aliases; the **same agent role on different hosts** also requires separate aliases. Do not publish actual hostnames, addresses, MACs, VM IDs, sensitive topology or hidden network mappings. Store any real-instance-to-public-alias mapping only in the secured private ledger after it is deployed. Until that mapping exists, describe the alias as `DECLARED_UNVERIFIED`, not authenticated.

Record `Writer-instance`, `Writer-role` and `Deployment-alias` on each contribution. For a non-Hermes writer use equivalent fields (e.g. `CHATGPT-REVIEW-01`, `REPOSITORY_AUDITOR`, `NOT_APPLICABLE`). Do not reuse an instance alias after it has been retired; log role or deployment changes without overwriting history. A separate publisher/relay may submit on behalf of the Hermes instance; name both and never assume the relay authored the content.

## Minimum mandatory Hermes machine identity

**Owner clarification (2026-11-06): Every Hermes instance MUST identify the machine on which that particular Hermes instance is running.** A generic agent name, security role, or GitHub login does not satisfy this minimum. Record:

- `Hermes-instance-ID`: stable unique identifier for the running Hermes instance, e.g. `HERMES-PRIVATE-02`.
- `Hermes-runtime-machine-ID`: stable, explicit **public-safe machine alias**, corresponding to the physical/virtual machine hosting that running Hermes process, e.g. `P2002` only if that label is already authorized for public release, otherwise `BOX-LAB-B`.
- `Hermes-role`: cloud/external, local/private, test or other actual role.
- `Target-machine-ID`: separate machine alias when Hermes operates on another machine. Do not conflate runtime host with a remotely managed target.

The machine ID is **mandatory** for Hermes-authored or Hermes-executed public activity; a placeholder such as `UNKNOWN` indicates a noncompliant, unverified record awaiting correction, **not acceptance**. If the runtime machine is not safely identified, **hold the publication** through the future gate and resolve it in the private ledger first. Where a public mapping itself would reveal sensitive topology, select a neutral one-to-one alias; never publish real hostname, MAC, hardware serial, private IP, VM UUID, or a cross-zone topology map. Distinct Hermes copies may share one runtime machine ID but MUST have different instance IDs. A Hermes instance moved to another machine must retain its instance identity while recording the machine change as a new event; historical events keep the original runtime machine ID.

Illustrative attribution (not a declaration of deployed instances):

```text
Writer-instance: HERMES-PRIVATE-02
Writer-role: PRIVATE_LOCAL_AGENT
Hermes-runtime-machine-ID: BOX-LAB-B
Target-machine-ID: BOX-LAB-A
Prepared-by: Hermes HERMES-PRIVATE-02 (declared)
Attribution-confidence: DECLARED_UNVERIFIED
```

This is an explicit **machine-of-execution minimum**, independent of GitHub identity. It remains a documented requirement until issue #6 and the publication gate (#1) enforce it.

## Required declarations for every new contribution

- **Writer-instance / writer-role / deployment-alias:** unique declared writer instance, security/task role and safe box/environment alias (or `NOT_APPLICABLE`). For Hermes all three are mandatory; see above.
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

- Writer-instance: CHATGPT-REVIEW-01 (declared; not authenticated)
- Writer-role: REPOSITORY_AUDITOR
- Deployment-alias: NOT_APPLICABLE
- Prepared-by: ChatGPT (this repository-audit conversation; model GPT-6, declared)
- Executed-by: NOT_APPLICABLE (documentation only)
- Evidence-source: TOOL_OBSERVED (GitHub repository files and commits)
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: owner request in this conversation (publication of attribution requirement)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: NOT_APPLICABLE
