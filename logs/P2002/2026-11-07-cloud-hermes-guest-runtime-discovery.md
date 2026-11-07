# P2002 — Cloud Hermes guest runtime discovery (read-only)

- **Checkpoint ID:** CLARYEL-P2002-20261107-GUEST-RUNTIME-DISCOVERY-001
- **Target host:** P2002
- **Guest VM:** `claryel-cloud-hermes`
- **Guest hostname:** `claryel-cloud-vm`
- **Guest user:** `claryelvm`
- **Guest address:** `172.30.240.12/24` (DHCP lease on `claryel-cloud-admin`)
- **Date:** 2026-11-07 (owner-supplied terminal output; timezone/time as displayed by guest)
- **Operation:** READ_ONLY_DIAGNOSTICS
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_OUTPUT + TOOL_OBSERVED_PUBLIC_REPOSITORY_REVIEW
- **Executed by:** Owner on P2002 and inside guest; ChatGPT only reviewed/publication
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-REVIEW-01
- **Writer-role:** REPOSITORY_AUDITOR
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED

## Access and guest identity

The owner connected through the existing ASUS jump host to P2002, then used SSH to reach `claryelvm@172.30.240.12`. The SSH login succeeded. Guest login banner reports Ubuntu 24.04.5 LTS, kernel `6.8.0-147-generic`; hostname is `claryel-cloud-vm`.

The host-side DHCP lease reported:
- MAC: `52:54:00:40:bd:81`
- IPv4: `172.30.240.12/24`
- DHCP hostname: `claryel-cloud-vm`

The QEMU Guest Agent is **not connected** (`virsh domifaddr --source agent` reports that it is not responding). This does not block SSH; the guest IP was obtained from DHCP lease and SSH access succeeded.

## Owner's existing-agent assertion

The owner explicitly states that Hermes is already installed inside this VM, connected and authorized with Grok, and has passed response tests. Do not reinstall Hermes, replace the guest, reset its state, or repeat provider authorization.

This is classified as **OWNER_REPORTED**. The public P2002 repository snapshot reviewed for this checkpoint contains the original VM provisioning chronology and the later owner correction, but no installation transcript identifying the Hermes binary/package, version, installation directory, runtime/service name, Grok connection evidence, or response-test transcript. The adjacent `ViktorIXAI/claryel-autonomous` repository contains benchmark experiment artifacts that mention Hermes as a benchmark agent, but those are not evidence that the Cloud Hermes guest was provisioned with the user's Grok-connected instance.

## Owner-supplied read-only guest diagnostics

The owner ran filtered checks for systemd user/system services and processes matching `hermes` or `nous`, plus the user's linger setting. The visible output was:

```text
=== IDENTITY ===
claryel-cloud-vm
claryelvm

=== USER SERVICES ===

=== RUNNING SERVICES ===

=== SYSTEM SERVICES ===

=== PROCESSES ===

=== USER LINGER ===
Linger=no
```

Interpretation:
- No matching Hermes/Nous system service unit was listed by the filtered query.
- No matching Hermes/Nous user service unit was listed by the filtered query.
- No matching Hermes/Nous process was returned by `pgrep -aif '[h]ermes|[n]ous'`.
- `loginctl show-user claryelvm -p Linger` returned `Linger=no`.
- These checks **do not prove that Hermes is not installed**. They do show that no process with those matching names was running at the time and no matching service unit was found by those filters.
- A CLI may be installed but not running; the runtime may have another name; it may be managed by Docker/Podman, a supervisor, tmux, cron, or a custom wrapper. These possibilities have not yet been checked.

## Repository consistency review

The current public P2002 initial-install narrative describes Cloud Hermes VM provisioning and guest OS setup, but the inspected text does not record a Hermes/Grok installation event. Issue #8's execution sequence also still contains earlier planned wording to install/start Hermes. The owner correction has since superseded that plan. This is a documentation/evidence gap, not proof that the owner's reported installation did not happen.

## Result

- **PASS:** SSH access to the existing guest.
- **PASS:** Guest identity and DHCP address established.
- **NOT_CONNECTED:** QEMU Guest Agent.
- **NOT_FOUND_BY_FILTER:** Hermes/Nous process and service names.
- **NOT_TESTED:** whether a Hermes CLI/package exists under another name or path.
- **OWNER_REPORTED:** Hermes installation, Grok authorization and response tests already completed.
- **NO_CHANGES:** no package, service, network, VM, credential or guest configuration was changed.

## Next action — discovery only, no reinstall

Run one grouped read-only inventory inside the guest to discover existing agent installation footprints:
1. Docker/Podman containers and images.
2. Executables and package-manager entries (`command -v`, pipx/uv/npm where installed).
3. Hermes-related directories under `/home/claryelvm`, `/opt`, `/usr/local` and systemd unit directories.
4. All running services and relevant process command lines, not only names containing Hermes.
5. Existing user shell startup, cron and supervisor/tmux entries.

Do not print environment variable values, tokens, API keys, provider credential files, browser profiles, or secret-bearing config contents. Only filenames, package names, unit names, process metadata and credential-presence booleans may be reported. Once the actual installation and launch mechanism are identified, inspect/enable that existing mechanism for boot; do not guess a unit name.



## Recovered prior installation record — Hermes v0.23.0 and Grok (2026-11-06)

A targeted retrieval of the prior conversation recovered the following historical setup facts. This is classified as **PRIOR_CONVERSATION_RECOVERED**, not a fresh guest-side command transcript:

- Hermes version previously reported: `0.23.0`.
- Installation directory: `/home/claryelvm/.hermes/hermes-agent`.
- CLI path: `/home/claryelvm/.local/bin/hermes`.
- Existing configuration paths: `/home/claryelvm/.hermes/config.yaml` and `/home/claryelvm/.hermes/.env`. File contents and credential values are intentionally not published.
- xAI OAuth was previously configured successfully; selected model was `grok-4.6`.
- Configured API base URL: `https://api.x.ai/v1`.
- Historical test command `hermes chat --oneshot` completed successfully using `grok-4.6`.

The recovered prior work explicitly did **not** document a persistent service, container, or boot-autostart mechanism. This explains why the current guest diagnostic can show no running Hermes process and no Hermes-named systemd unit even though the CLI and configuration were previously installed. The owner-supplied filtered checks are consistent with a previously tested one-shot CLI rather than a persistent agent service.

### Updated conclusion

- Existing Hermes installation and Grok authorization: **PRIOR-CONVERSATION_RECOVERED**.
- Previous Grok one-shot response test: **PRIOR-CONVERSATION_RECOVERED / reported successful**.
- Current Hermes process: **NOT_RUNNING in the owner-supplied snapshot**.
- Existing persistent service/autostart: **NOT_DOCUMENTED / not found by the filtered unit query**.
- Reinstall or reauthorize: **NOT REQUIRED**.
- Next step: inspect the existing CLI's supported persistent/gateway mode and create the correct boot-start unit for the existing installation, preserving its home directory and credential files. First inspect CLI help and existing files/permissions; do not print or copy secret values.

The credential object identifier and all OAuth token material are intentionally excluded from this public log.


## Owner-supplied follow-up guest check — 2026-11-07 03:29 UTC

- **Checkpoint ID:** CLARYEL-P2002-20261107-GUEST-PATH-CHECK-001
- **Operation:** READ_ONLY_DIAGNOSTICS
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner, through SSH from P2002 into the existing guest; ChatGPT did not execute commands on either machine.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Deployment-alias:** P2002-LAB
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (read-only diagnostics)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-checkpoint:** CLARYEL-P2002-20261107-GUEST-RUNTIME-DISCOVERY-001

The owner successfully SSH-connected from P2002 to the existing guest. The guest reported hostname `claryel-cloud-vm` and kernel `6.8.0-147-generic`.

The command used `command -v hermes`, which reported `Hermes CLI not found in PATH`. This is **not evidence that Hermes is absent**: the previously recovered installation record identifies the executable at the absolute path `/home/claryelvm/.local/bin/hermes`, which may not be included in the non-interactive SSH command's PATH.

The following results were visible:
- No matching system service unit found by the Hermes/Nous/Gateway name filter.
- No matching user service unit found by the same filter.
- No running Docker or Podman containers were listed.
- No matching Hermes/Nous/Gateway process was listed.
- `loginctl show-user "$USER" -p Linger` returned `Linger=no`.
- No matching cron entry was found.
- Permission-only check confirmed `~/.hermes` is mode `700`, and `config.yaml` and `.env` are mode `600`. No file contents or credential values were displayed.

### Result

- **PASS:** SSH access to the existing guest.
- **OBSERVED:** Hermes is not found by the default PATH lookup in this non-interactive SSH session.
- **NOT_TESTED:** the known absolute CLI path, actual Hermes version from the guest, and supported Gateway command options.
- **NOT_FOUND_BY_FILTER:** Hermes-named system/user units and running process.
- **NOT_FOUND:** matching guest containers and cron entry in the displayed checks.
- **OBSERVED:** user lingering is disabled.
- **PASS (permissions only):** expected restrictive permissions on the existing Hermes config directory and files.
- **NO_CHANGES:** no package, service, credential, configuration, VM, or network changes were made.

### Next action

Use the known absolute executable path `/home/claryelvm/.local/bin/hermes` for read-only `--version`, top-level `--help`, and `gateway --help` inspection. Do not reinstall, reauthorize, print secret-bearing configuration, or create a service until the actual supported persistent Gateway invocation is confirmed.
