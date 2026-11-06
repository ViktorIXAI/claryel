# CLARYEL audit checkpoint template (synthetic/example only)

> Follow policies/PUBLICATION-SECURITY.md. Do not paste real terminal output, addresses, hostnames, identifiers or secrets into a public commit. Record private full-fidelity details in protected evidence storage **only once that storage is deployed**. This template does not authorize execution.

- **Checkpoint ID:** CLARYEL-BOX-YYYYMMDD-NNN (unique; immutable)
- **Event time (UTC):** UNKNOWN / YYYY-MM-DDTHH:MM:SSZ (actual execution time, not commit time)
- **Publication time (UTC):** UNKNOWN / YYYY-MM-DDTHH:MM:SSZ
- **Target role:** BOX_TEST_HOST (no actual hostname or IP)
- **Writer-instance (persistent declared alias, unique for each Hermes):** UNKNOWN
- **Writer-role (e.g. PUBLIC_CLOUD_AGENT / PRIVATE_LOCAL_AGENT):** UNKNOWN
- **Hermes-runtime-machine-ID (MANDATORY for every Hermes instance; safe physical/VM host alias):** NOT_APPLICABLE for non-Hermes / required safe machine ID for Hermes
- **Target-machine-ID (separate remote target, if any):** NOT_APPLICABLE / safe target alias
- **Deployment-alias (safe box/environment alias, never real hostname/IP):** NOT_APPLICABLE / UNKNOWN
- **Prepared-by (person/model/agent or safe chat alias):** UNKNOWN
- **Executed-by (actual machine operator, if any):** NOT_APPLICABLE / UNKNOWN
- **Evidence-source:** OWNER_REPORTED | TOOL_OBSERVED | SIMULATED | INDEPENDENTLY_VERIFIED | PROPOSED | UNKNOWN
- **Reviewed-by:** NOT_REVIEWED / SELF_REVIEW_ONLY / independently reviewed
- **Approved-by:** NOT_CONFIRMED / documented owner approval
- **Published-by:** shared credential AND submitting agent/process if known
- **Attribution-confidence:** DECLARED_UNVERIFIED | VERIFIED
- **Approval reference:** UNKNOWN / reviewed owner decision ID
- **Purpose / expected behavior:**
- **Preconditions / safety and privacy impact:**
- **Operation status:** PROPOSED | APPROVED | EXECUTED | ABORTED | NOT_TESTED
- **Sanitized exact commands or versioned script reference:**
- **Command exit code(s):** UNKNOWN / observed numeric status (never invent)
- **Observed output (sanitized):**
- **Validation method and independent verifier:**
- **Result:** PASS | FAIL | UNKNOWN | NOT_TESTED (PASS applies only to tested criterion)
- **Limitations, assumptions and negative evidence:**
- **Rollback / recovery status:** NOT_TESTED / describe verified result
- **Public sanitized artifact digest (SHA-256):** UNKNOWN / digest of reviewed public artifact only
- **Private evidence checkpoint mapping:** NOT_DEPLOYED / opaque ID only, never sensitive path or hash of low-entropy secret
- **Related issues, commits, test fixtures and version pins:**
- **Next action (not authorization):**

## Review checklist

- [ ] Execution context and actor are evidenced, not inferred from Git author identity.
- [ ] Each Hermes instance has distinct stable alias, role and required runtime machine ID (same or different boxes); record remote target separately. Missing Hermes runtime machine ID blocks publication once gate is enforced.
- [ ] Each test has a declared scope; simulated test is not relabeled live validation.
- [ ] Missing evidence is labeled UNKNOWN or NOT_TESTED.
- [ ] Public output is synthetic or independently sanitized before upload.
- [ ] Publication gate ran on exact outgoing bytes; absent gate is explicitly disclosed.
- [ ] No other agent's live installation log was modified or reordered.

**Example use:** Copy to a **new** event document after review; never use this template as a claim of execution.
