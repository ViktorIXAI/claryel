# P2002 — Existing Cloud Hermes guest agent: owner correction and next action

- **Checkpoint ID:** CLARYEL-P2002-20261107-GUEST-AGENT-CORRECTION-001
- **Target-machine-ID:** P2002
- **Guest VM:** `claryel-cloud-hermes`
- **Guest agent:** Hermes (existing installation; do not reinstall)
- **External provider:** Grok (owner-reported connected and authorized)
- **Evidence source:** OWNER_REPORTED for installed/configured/tested agent; TOOL_OBSERVED for repository status
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Executed-by:** NOT_APPLICABLE (documentation only)
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED

## Owner correction

The owner explicitly states that Hermes is already installed inside the existing Cloud Hermes VM, connected to Grok, authorized with the provider, and has already passed response tests.

This is an owner-reported completion, not an independently observed runtime result in this ChatGPT session. The public repository files and issue comments inspected for this checkpoint still described Hermes installation and Grok connection as future work. The original installation/test transcript, exact Hermes version, service-unit name and authorization mechanism were not present in the inspected public evidence. No secrets or credentials are included here.

## Operational consequence

- Do **not** install Hermes again.
- Do **not** replace the current guest, reset its state, or repeat Grok authorization.
- The next task is to inspect the existing Hermes service/process inside the guest and establish whether it is enabled for automatic startup.
- If required, enable the already-installed service using the actual unit name discovered from the guest; never guess the name.
- Then continue the planned integration of this existing Hermes instance with the working private local inference API.
- Keep the current libvirt network, Guard and inference isolation intact. Use synthetic test data until the API authentication/Security Gateway boundary is completed.

## Host-side progress recorded separately

The owner has just enabled autostart for both:
- libvirt network `claryel-cloud-admin`;
- VM `claryel-cloud-hermes`.

See [the host-side autostart checkpoint](2026-11-07-cloud-hermes-autostart.md). This does not prove the guest Hermes service itself is enabled or that a post-reboot guest/agent startup has been tested.

## Evidence still needed

Recover or capture a sanitized guest-side transcript showing:
1. Existing Hermes version/runtime identity (if readily available).
2. Actual service/process name and status.
3. Whether the service is enabled at boot.
4. Existing Grok provider connection status without exposing tokens, keys, account identifiers or secret values.
5. The already-completed response-test evidence, if retained.

Do not delay the integration sprint for broad security audits. Do not claim independent verification until the corresponding output is available.
