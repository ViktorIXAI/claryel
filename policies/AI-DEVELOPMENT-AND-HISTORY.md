# AI-Assisted Development, Fallibility, and Historical Integrity Policy

**Status:** binding project governance principle, effective 2026-11-06.

## Authorship and accountability

CLARYEL is developed and documented with substantial involvement from AI systems. Public repository changes may be proposed, written, reviewed, or published by **ChatGPT**, **Grok**, and, as the deployment evolves, primarily by the locally configured **Hermes agents and local AI system**. Other AI agents, models, tools, and human contributors may join later.

This is a statement of the **intended collaborative workflow**, not a claim that all named agents have already made commits or have ongoing publishing access. For each actual change, distinguish the proposing agent, author/operator, reviewer, publisher and accountable human approver where reliably known. Do not invent provenance or attribute GitHub account activity to a model without evidence.

The human project owner, participating humans, and every AI system are **fallible**. Expect misunderstandings, omissions, incorrect assumptions, unsafe defaults, faulty tests, fabricated claims, code defects and unexpected behavior. A previous successful deployment, an earlier review, an authoritative-sounding explanation or a consensus of multiple models is never sufficient proof of correctness. This applies both when reproducing previously tested solutions from scratch and when inventing new solutions.

## Evidence-first, step-by-step development

- Build incrementally from a fresh, minimally privileged baseline. Each step requires a documented hypothesis, threat assessment, explicit owner-controlled change scope, independent verification, measured outcome, residual risk and rollback strategy.
- Existing CLARYEL or third-party code/configuration is **untrusted reference material**. Consult it to learn from experience, but do not silently reuse code, permissions, secrets, assumptions or security claims; review dependencies and attack surfaces afresh.
- Record negative results, failed hypotheses, regressions, abandoned decisions and unresolved disagreements just as carefully as successes.
- Review agent-generated changes with tests and human checkpoints appropriate to the risk; agents may not approve their own privilege escalations, security policy changes, secret disclosure or deployment.
- No public issue, model output, pull request or comment by itself grants permission to execute actions on a host or expose data.
- Never describe a proposed control as deployed, a reported test as independently verified, or a design target as achieved compliance.

## Preserve history for future analysis

The repository's evolution is itself research evidence. Preserve the chronological history of proposals, revisions, mistakes, fixes, tests, reviews and decisions, including human and AI mistakes.

- Prefer additive corrections with explicit links to superseded claims. Preserve original records and dated context rather than silently rewriting historical claims or benchmark evidence.
- Do not force-push, rebase published history, delete evidence branches/tags, or run destructive cleanup solely to hide errors. Corrections should be traceable to prior versions and their rationale.
- Use reviewable issues/PRs and descriptive commits with references to test evidence. Include change author/provenance where supportable, timestamps, assumptions, test scope, outcome and uncertainty.
- GitHub history is **not a sufficient backup**: keep independently verified local mirrors and immutable snapshots/bundles, protect against deleted or force-pushed refs, and periodically test offline restoration. Treat issues, PR discussions, Actions artifacts, LFS and releases as separate backup categories.
- Preserve research evidence **without publishing secrets, personal data, client content, privileged network details or exploitable operational configurations**. If sensitive information is exposed, prioritize containment, credential revocation and legally required remediation; authorized removal/redaction may be necessary and overrides ordinary non-deletion rules. Keep an appropriately secured, access-restricted incident audit record, not a public copy of the sensitive payload.
- Retention and publication practices must comply with applicable privacy, contractual and legal obligations, including deletion and minimization requirements. **Do not retain personal data indefinitely merely to preserve research history.**

## Model and agent diversity

Treat agreement among ChatGPT, Grok, Hermes or any other models as additional review input, never independent validation by default. Where possible use reproducible deterministic tests and threat-based negative cases. Identify shared prompts, source material and correlated failure modes when comparing model judgments.

## Publication

Everything placed in the public repository—including documentation, commit messages, issue/PR text, reviews and agent reports—**must be in English**, subject to the [publication-language policy](PUBLICATION-LANGUAGE.md). Public evidence must be synthetic or safely redacted and must pass human-supervised disclosure checks before publication.

**Operating principle:** Assume we will all make mistakes. Make mistakes discoverable, reversible and analyzable, without turning the public repository into a channel for leaking protected information.
