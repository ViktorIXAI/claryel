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


## Owner-supplied Hermes CLI and Gateway capability check — 2026-11-07 03:29 UTC

- **Checkpoint ID:** CLARYEL-P2002-20261107-HERMES-GATEWAY-CLI-001
- **Operation:** READ_ONLY_CLI_INSPECTION
- **Target-machine-ID:** P2002-CLOUD-GUEST
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner, in the already-open guest shell; ChatGPT did not execute commands on the guest.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Deployment-alias:** P2002-LAB
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (read-only CLI inspection)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoint:** CLARYEL-P2002-20261107-GUEST-PATH-CHECK-001

The owner invoked the already-installed Hermes executable by its known user-local absolute path (not relying on the non-interactive shell PATH).

### Observed CLI version

```text
Hermes Agent v0.23.0 (2026.9.24) · stable · local 2bd378ed1b8d
Install method: git
Python: 3.14.9
OpenAI SDK: 2.54.0
Up to date
```

The installation reported its existing Git installation directory; this record intentionally does not repeat the guest's absolute home-directory path.

### Observed Gateway command surface

The top-level CLI help exposes `gateway` as the messaging gateway management command. Its supported subcommands are:

- `run` — run the gateway in the foreground (CLI help describes this as recommended for WSL and Docker).
- `ensure` — ensure a local runtime without installing or replacing a service.
- `start`, `stop`, `restart`, `status`.
- `install` — install the gateway as a systemd/launchd background service.
- `uninstall`, `list`, `setup`, `migrate-legacy`, `migrate`, `enroll`.

The presence of `gateway install` is a supported CLI capability. It is **not** evidence that a service has already been installed or enabled. The owner has not yet run the installer.

### Result

- **PASS:** Existing Hermes CLI is executable when invoked by its known absolute path.
- **PASS:** Hermes version is v0.23.0, stable, Git installation.
- **PASS:** CLI explicitly supports a Gateway systemd/launchd service installation command.
- **NOT_TESTED:** Current Gateway service status from Hermes' own status command.
- **NOT_TESTED:** Exact install options, service scope, unit name, systemd user-manager requirements, and behavior when user lingering is disabled.
- **NO_CHANGES:** No Gateway installation, start/stop, service, user-linger, package, configuration, credential, VM, or network changes were made.

### Next action

Run read-only `gateway status`, `gateway install --help`, `gateway ensure --help`, and `gateway run --help` using the existing CLI. Determine the service scope and exact installer behavior before approving any installation. Preserve the existing Grok authorization and Hermes home/configuration. Do not invoke `gateway install` yet.


## Owner-supplied Gateway status and installer option check — 2026-11-07

- **Checkpoint ID:** CLARYEL-P2002-20261107-GATEWAY-INSTALLER-OPTIONS-001
- **Operation:** READ_ONLY_GATEWAY_AND_SYSTEMD_DIAGNOSTICS
- **Target-machine-ID:** P2002-CLOUD-GUEST
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner, in the already-open guest shell; ChatGPT did not execute commands on the guest.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Deployment-alias:** P2002-LAB
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (read-only service diagnostics)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoint:** CLARYEL-P2002-20261107-HERMES-GATEWAY-CLI-001

### Observed Gateway status

The existing Hermes CLI returned:

```text
✗ Gateway is not running

Recent gateway health:
  ⚠ Last shutdown reason: auto-started gateway idle for 600s (gateway.unmanaged_idle_exit_seconds)

To start:
  hermes gateway run
  hermes gateway install
  sudo hermes gateway install --system
```

Interpretation: the Gateway was not running at the time of this check. Its recorded recent shutdown reason is the configured 600-second unmanaged-idle exit. This is not, by itself, evidence of a crash or failed installation. It does confirm there is currently no running Gateway process according to the CLI status command.

### Installer options observed

`hermes gateway install --help` lists:
- `--system`: install as a Linux system-level service (help text says starts at boot).
- `--run-as-user RUN_AS_USER`: select the account used by the Linux system service.
- `--start-now` / `--no-start-now`.
- `--start-on-login` / `--no-start-on-login`.
- `--if-missing`, `--force`, and `--force-unit-path`.

The Gateway `ensure` command is documented as ensuring a local runtime without installing or replacing a service. The CLI reports readiness-related exit codes 0–7.

The Gateway `run` help documents foreground operation, including `--replace`, `--external-supervisor`, and options that apply to Docker/s6. No Gateway command was executed; only help and status were read.

### systemd observation

`systemctl --user is-system-running` returned `running`. The user service-unit filter returned no Hermes unit. Earlier owner-supplied diagnostics reported `Linger=no`. A user systemd manager being currently active does not prove that user services will start at boot without a login; with lingering disabled, that boot behavior must not be assumed.

### Result

- **PASS:** Hermes CLI reports Gateway not running at check time.
- **OBSERVED:** recent Gateway shutdown reason was the configured unmanaged idle exit after 600 seconds.
- **PASS:** CLI supports installing a Linux system-level boot service and selecting its run-as account.
- **PASS:** user systemd manager is running in the current session.
- **NOT_FOUND_BY_FILTER:** no Hermes user unit was listed.
- **NOT_TESTED:** system-level service inventory, exact generated unit contents, installer privilege requirements, runtime environment, restart policy, and actual boot behavior.
- **NO_CHANGES:** no service was installed, enabled, started, stopped, or modified.

### Next action

Before invoking the installer, inspect the local v0.23.0 Gateway installer implementation and exact systemd unit-generation behavior (including service identity, HOME/HERMES_HOME, working directory, environment handling, restart policy, and whether `--system` requires root). Also check for pre-existing Hermes system-level units. Then use the supported system-level installer for account `claryelvm` only if the implementation preserves the existing Hermes home and credential access. Do not use `--force` or `--force-unit-path` unless a verified conflict requires it. No provider reauthorization or reinstall is permitted.


## Owner-supplied installer-source and existing-unit inventory — 2026-11-07

- **Checkpoint ID:** CLARYEL-P2002-20261107-GATEWAY-INSTALLER-SOURCE-SEARCH-001
- **Operation:** READ_ONLY_SOURCE_AND_SYSTEMD_INVENTORY
- **Target-machine-ID:** P2002-CLOUD-GUEST
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner, in the already-open guest shell; ChatGPT did not execute commands on the guest.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Deployment-alias:** P2002-LAB
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (read-only source and unit inspection)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoint:** CLARYEL-P2002-20261107-GATEWAY-INSTALLER-OPTIONS-001

### Commands and observed output

The owner searched Python source files in the existing Hermes Git installation for the Gateway/systemd installer identifiers and related unit-generation terms, then queried installed system service units and searched standard system unit directories for Hermes/Nous unit files. The broad source search was piped through `head -160`; its output was dominated by generic systemd references and did not identify the Gateway install implementation. The expanded per-user source path is intentionally not repeated in this public log.

The system unit inventory produced no matching service-unit entries. The standard system unit-file search produced no matching Hermes/Nous unit files.

One source comment was visible in the search output at `agent/agent_init_memory_provider.py`: it states that systemd Gateway services do not automatically inherit the Hermes home `.env` file and that the relevant environment must be set separately. This is a source-code warning about environment propagation; the truncated search excerpt does not establish the exact installer behavior or which individual credentials/settings are needed by this particular service. It is a reason to inspect the relevant implementation before installation, not proof that Grok authorization will be lost.

### Result

- **PASS:** No existing Hermes/Nous system-level service unit was found by the supplied systemd unit-file query.
- **PASS:** No Hermes/Nous-named unit file was found by the supplied standard unit-directory search.
- **OBSERVED:** The broad Python source search did not locate or expose enough of the Gateway installer implementation to establish its generated unit semantics.
- **IMPORTANT SOURCE WARNING:** A Hermes source comment explicitly warns that systemd Gateway services do not automatically inherit the Hermes home `.env` file.
- **NOT_TESTED:** Exact Gateway installer source, generated unit content, service environment, credential loading, restart policy, and runtime identity.
- **NO_CHANGES:** No service, package, configuration, credential, VM, or network changes were made.

### Next action

Perform a targeted read-only source search for the exact Gateway installer implementation and unit writer (terms such as `gateway install`, `run_as_user`, `start_on_login`, `EnvironmentFile`, `HERMES_HOME`, and the generated `hermes-gateway` unit). Read only the relevant source sections and any directly called unit-generation helper. Establish the exact environment/credential behavior before invoking the installer. Do not print `.env` contents, config values, tokens, or secrets. Do not use force/repoint flags, reinstall Hermes, or reauthorize Grok.


## Owner-supplied installed-source search and exact-version upstream review — 2026-11-07

- **Checkpoint ID:** CLARYEL-P2002-20261107-GATEWAY-SYSTEMD-SOURCE-REVIEW-001
- **Operation:** READ_ONLY_SOURCE_REVIEW_AND_INSTALL_PLAN
- **Target-machine-ID:** P2002-CLOUD-GUEST
- **Deployment-alias:** P2002-LAB
- **Event time:** UNKNOWN (the supplied terminal output contains no execution timestamp)
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT + TOOL_OBSERVED_UPSTREAM_SOURCE_AT_EXACT_COMMIT
- **Executed-by:** Owner ran the local source-version/search commands in the existing guest shell. ChatGPT did not execute commands on the guest.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (read-only inspection and continuation of installation planning)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoint:** CLARYEL-P2002-20261107-GATEWAY-INSTALLER-SOURCE-SEARCH-001

### Owner-supplied local source identity

The owner supplied this read-only output from the existing Hermes checkout:

```text
git rev-parse --short HEAD
2bd378ed1b

git describe --tags --always
v0.23.0
```

The broad grep searched Python source files (excluding tests and the virtual environment) for Gateway installation, systemd, environment, and service-unit terms. Its first 200 matches did not expose the installer implementation. It did reveal relevant source locations, including `hermes_cli/gateway_service_owner.py`, `hermes_cli/gateway.py`, `hermes_cli/subcommands/gateway.py`, and a scheduler note concerning the user systemd manager. The output was truncated by `head -200`; it must not be treated as a complete source audit.

The reported short commit was then resolved through the upstream NousResearch/hermes-agent GitHub repository to the full commit `2bd378ed1b8d6bf2192cb26e0a55ff3091e6bd19`. The source review below is pinned to that exact commit, not current upstream main.

### Exact v0.23.0 source findings

1. **System service identity and home**
   - `gateway.py::generate_systemd_unit(system=True, run_as_user=...)` resolves the selected account and its primary group.
   - The generated unit sets `User=` and `Group=`, plus `HOME=`, `USER=`, and `LOGNAME=` for that account.
   - It explicitly pins `HERMES_HOME=` to that account's existing Hermes home and uses the corresponding Python executable, working directory and PATH.
   - The service command is the installed Hermes runtime's `gateway run`, not an unmanaged foreground command started from an interactive shell.
   - The generated system unit orders itself after and wants `network-online.target` and `user@<uid>.service`; it is wanted by `multi-user.target`.

2. **Restart and logging behavior**
   - The generated unit sets `Restart=always` and `RestartSec=5`.
   - It defines special exit handling for Hermes' planned restart and fatal configuration exit codes; therefore the policy is not simply an unconditional retry for every configuration failure.
   - It uses `KillMode=mixed`, `KillSignal=SIGTERM`, a SIGUSR1 reload/drain action, explicit stop/cleanup helpers, and sends standard output/error to the system journal.
   - These are source-defined unit properties; they have not yet been observed in a generated unit on the guest.

3. **Existing .env and provider authentication**
   - The generated systemd unit does not add an `EnvironmentFile=` directive for the Hermes home `.env`.
   - However, the exact-version CLI entry point `hermes_cli/main.py` calls `load_hermes_dotenv()` during startup. The exact-version `env_loader.py` loads the process Hermes home's `.env` and related configured secret sources.
   - The same CLI code checks provider login state in `HERMES_HOME/auth.json`. The service unit explicitly sets `HOME` and `HERMES_HOME` for the selected existing account.
   - Therefore the earlier source comment in `agent/agent_init_memory_provider.py` must be read narrowly: it warns that a systemd service does not inherit the home `.env` as its process environment for the external memory-provider availability check. It is **not evidence that Grok OAuth will be lost**. The CLI startup path itself loads the Hermes home dotenv, and the provider auth store is resolved under that same Hermes home.
   - No `.env`, `auth.json`, OAuth token, API key, or credential value was read or published. Preservation of the existing Grok login remains to be confirmed by a post-start status/response test.

4. **Root requirement and lingering side effect**
   - `systemd_install(system=True, ...)` explicitly requires root.
   - After creating/enabling the system unit, the installer calls `_ensure_system_service_linger()` for the configured service account. If linger is not already enabled, the installer attempts `loginctl enable-linger` and waits for the target user's D-Bus socket.
   - The source explains that this is needed for Gateway cron/Kanban workers that use `systemd-run --user`; it is not merely a workaround for boot-start.
   - Thus the previously observed `Linger=no` is not a blocker, but it is expected to change to enabled if this installer step succeeds. This side effect must be visible to the owner before execution.

5. **Install/start switches**
   - `--system` selects a system-level boot service.
   - `--run-as-user "$USER"` explicitly selects the currently logged-in guest account as the service identity; the CLI's source remaps the default Hermes home to that account.
   - `--start-on-login` enables the unit at boot (`multi-user.target`).
   - `--start-now` starts the newly installed service immediately after installation.
   - The source path contains ownership checks for an existing service definition. The prior guest inventory found no Hermes/Nous system service or matching unit file, so there is no observed reason to use `--force` or `--force-unit-path`.

### Result

- **PASS:** Owner-supplied local checkout reports v0.23.0 and short commit `2bd378ed1b`.
- **PASS:** Upstream source at the exact full commit was fetched and inspected.
- **CONFIRMED IN SOURCE:** service account, HOME/HERMES_HOME, systemd dependencies, restart policy, journal logging, root requirement, start/enable behavior, and installer-managed linger behavior described above.
- **CORRECTION:** the memory-provider warning does not establish loss of Grok OAuth; the CLI entry point loads its Hermes home dotenv and checks the provider auth store under HERMES_HOME.
- **NOT_TESTED:** actual generated unit, actual service start, Grok authentication from the Gateway process, restart after process failure, and boot after guest/host power cycle.
- **NO_CHANGES:** no service was installed, enabled, started, stopped, or modified; no credentials were read or changed.

### Proposed next owner operation — not yet executed

After reviewing the documented side effect that the installer may enable systemd linger for the service account, the owner can run this single command from the already-open guest shell:

```bash
sudo "$HOME/.local/bin/hermes" gateway install --system --run-as-user "$USER" --start-now --start-on-login
```

This is a proposed state-changing command, not evidence of execution or approval already exercised. It should create a boot-enabled system service running as the current non-root guest account, preserve the existing Hermes home and credential files, and start the Gateway now. Do not add `--force` or `--force-unit-path`. Capture the complete installer output. If the installer reports an error or a linger warning, stop and report it rather than improvising.

### Publication review note

The outgoing summary was manually reviewed against the active owner-authorized P2002 lab disclosure scope. It contains no credential values, private keys, customer data, or guest account/home-directory paths. A Gitleaks executable was not available to this connector writer, so no Gitleaks scan is claimed; the repository's future automated fail-closed publication gate remains unimplemented/unverified.

### Contribution provenance

- Writer-instance: CHATGPT-INSTALL-01 (declared; unauthenticated)
- Writer-role: INSTALLATION_COORDINATOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: Owner for the local read-only commands; ChatGPT did not execute on P2002
- Evidence-source: OWNER_SUPPLIED_TERMINAL_TRANSCRIPT + TOOL_OBSERVED_UPSTREAM_SOURCE_AT_EXACT_COMMIT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-07 (read-only source inspection and installation planning)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: CLARYEL-P2002-20261107-GATEWAY-SYSTEMD-SOURCE-REVIEW-001

## Owner-supplied Hermes PM repair failure — DNS resolution blocks PyPI

- **Checkpoint ID:** CLARYEL-P2002-20261107-PM-REPAIR-DNS-FAILURE-001
- **Target-machine-ID:** P2002-CLOUD-GUEST
- **Deployment-alias:** P2002-LAB
- **Event time:** UNKNOWN (no timestamp in the supplied terminal transcript)
- **Operation:** OWNER_EXECUTED_DEPENDENCY_REPAIR_ATTEMPT
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner, as the existing non-root guest account in the already-open VM shell; ChatGPT did not execute on the guest.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (owner-controlled repair attempt)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoint:** CLARYEL-P2002-20261107-GATEWAY-SYSTEMD-SOURCE-REVIEW-001

### Executed command

The owner ran hermes pm repair as the existing guest user (not via sudo), from the existing Hermes checkout.

### Observed failure

The PM repair process began and attempted to build the Hermes package in its recorded dependency environment. It failed while resolving the build-system requirements:

- setuptools==84.1.0
- wheel

The package index request failed at https://pypi.org/simple/setuptools/. The terminal reported three retries followed by:

    Temporary failure in name resolution

The PM command exited with dependency repair failure; the nested uv sync returned exit code 2. The terminal explicitly recommended retrying or running hermes pm doctor / hermes pm repair.

### Assessment

- **CONFIRMED:** The repair command was run as the checkout owner (UID 1000), avoiding the earlier root-versus-owner checkout permission conflict.
- **CONFIRMED:** The immediate blocker is name resolution for pypi.org during dependency installation.
- **NOT_YET_DETERMINED:** Whether the guest's resolver configuration/service is broken, its default route or upstream network is unavailable, or host-side Guard/egress policy is blocking DNS/HTTPS.
- **NO_CHANGES_CONFIRMED:** The supplied output shows a failed dependency repair attempt; it does not show successful environment repair or Gateway installation.
- **BLOCKED:** Do not run hermes update or gateway install until the dependency environment is repaired and verified.

### Next action

Run a read-only guest network diagnostic while remaining in the existing VM shell:
- interface addresses and default route;
- /etc/resolv.conf and resolvectl status;
- getent ahostsv4 pypi.org and resolvectl query pypi.org;
- reachability of the configured default gateway;
- HTTPS HEAD request to the setuptools package index.

Do not change DNS servers, edit resolver files, open egress, disable Guard, alter nftables, or change libvirt networking until the diagnostics identify the failure layer and the owner approves a specific controlled change.

### Contribution provenance

- Writer-instance: CHATGPT-INSTALL-01 (declared; unauthenticated)
- Writer-role: INSTALLATION_COORDINATOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: Owner in the existing guest shell
- Evidence-source: OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-07 (owner-controlled repair attempt)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: CLARYEL-P2002-20261107-PM-REPAIR-DNS-FAILURE-001

## Owner-supplied guest network diagnosis — no default route or DNS uplink

- **Checkpoint ID:** CLARYEL-P2002-20261107-GUEST-NETWORK-NO-DEFAULT-ROUTE-001
- **Target-machine-ID:** P2002-CLOUD-GUEST
- **Deployment-alias:** P2002-LAB
- **Event time:** UNKNOWN (no timestamp in the supplied guest transcript)
- **Operation:** READ_ONLY_NETWORK_DIAGNOSTICS
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner, in the already-open guest shell; ChatGPT did not execute commands on the guest.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (read-only network diagnosis)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoint:** CLARYEL-P2002-20261107-PM-REPAIR-DNS-FAILURE-001; CLARYEL-P2002-20261107-HOST-BOOT-SNAPSHOT-001

### Owner-supplied observations

- Interface `enp6s0` is UP with address `172.30.240.12/24`.
- The only route shown is the connected subnet route `172.30.240.0/24 dev enp6s0`; there is **no default route**.
- `/etc/resolv.conf` is a symlink to the systemd-resolved stub file and lists only `127.0.0.53`.
- `systemd-resolved` is active, but `resolvectl status` reports no DNS scopes on `enp6s0` and no configured uplink DNS server.
- `getent ahostsv4 pypi.org` produced no address.
- `resolvectl query pypi.org` failed with "No appropriate name servers or networks for name found".
- The HTTPS request to the PyPI package index failed with curl error 6, "Could not resolve host".
- The supplied transcript contains an output splice/truncation after the "DEFAULT GATEWAY TEST" heading; no successful gateway ping result is present. The route table independently establishes that no default gateway route is configured.

### Correlation with earlier host evidence

The earlier owner-supplied P2002 host snapshot identified libvirt network `claryel-cloud-admin`, bridge `virbr240`, and the Guard nftables table `inet claryel_guard`. Its displayed `guest_forward` chain explicitly drops traffic entering or leaving `virbr240`; its `guest_input` chain allows established/related traffic and DHCP before dropping other traffic arriving on that bridge.

These host rules are evidence of an intentional isolation boundary. They do not, by themselves, identify the intended approved egress path for this guest or prove whether a separate proxy path is available. The guest's missing default route and DNS uplink are independently observed.

### Assessment

- **CONFIRMED:** The guest currently has no default route.
- **CONFIRMED:** The guest has no DNS uplink/scope configured on `enp6s0`.
- **CONFIRMED:** PyPI hostname resolution and HTTPS access fail from the guest.
- **LIKELY:** The immediate failure is due to the guest's isolated network configuration and/or intentionally restricted egress, not a Hermes-specific issue.
- **NOT_YET_DETERMINED:** The intended route for this guest (direct routing, restricted proxy, Security Gateway relay, or another approved path), the libvirt network's exact DHCP/router configuration, and the host-side proxy policy.
- **NO_CHANGES:** No DNS settings, routes, Guard rules, libvirt network settings, or guest configuration were changed.

### Next action

On the P2002 host (not inside the guest), perform read-only inspection of:
1. `virsh net-dumpxml claryel-cloud-admin`;
2. bridge address and link state for `virbr240`;
3. host routes relevant to `172.30.240.0/24`;
4. the relevant Guard/proxy service status and narrowly scoped rules/counters;
5. any already-defined approved Security Gateway or proxy path for this guest.

Do not add a default route, assign public DNS, enable libvirt forwarding, relax nftables, or disable Guard until the intended security architecture path is identified and explicitly approved by the owner.

### Contribution provenance

- Writer-instance: CHATGPT-INSTALL-01 (declared; unauthenticated)
- Writer-role: INSTALLATION_COORDINATOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: Owner in the existing guest shell
- Evidence-source: OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-07 (read-only network diagnosis)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: CLARYEL-P2002-20261107-GUEST-NETWORK-NO-DEFAULT-ROUTE-001

## Owner-supplied P2002 host network and proxy inspection — isolated Cloud Hermes segment

- **Checkpoint ID:** CLARYEL-P2002-20261107-HOST-NETWORK-PROXY-TOPOLOGY-001
- **Target-machine-ID:** P2002
- **Deployment-alias:** P2002-LAB
- **Event time:** UNKNOWN (no timestamp in the supplied host transcript)
- **Operation:** READ_ONLY_HOST_NETWORK_AND_PROXY_DIAGNOSTICS
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner on P2002 host; ChatGPT did not access or execute commands on the host.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (read-only host diagnostics)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoint:** CLARYEL-P2002-20261107-GUEST-NETWORK-NO-DEFAULT-ROUTE-001; CLARYEL-P2002-20261107-HOST-BOOT-SNAPSHOT-001

### Owner-supplied libvirt network evidence

The owner ran `virsh net-dumpxml claryel-cloud-admin`. The returned network definition contains:
- Network name: `claryel-cloud-admin`.
- Bridge: `virbr240`.
- Host bridge address: `172.30.240.1/24`.
- DHCP range: `172.30.240.10`–`172.30.240.20`.
- No `<forward ...>` element is present in the XML. This is consistent with a libvirt isolated network, not a routed/NAT network.

Host bridge state:
- `virbr240` is UP with `172.30.240.1/24`.
- Guest interface `vnet0` is attached and forwarding at layer 2.
- The host route table contains the connected route `172.30.240.0/24 dev virbr240`.
- Host route lookup for `172.30.240.12` resolves directly via `virbr240`.

Host default route is via `192.168.1.1` on `enp128s31f6`. This is a host route; it does not create a default route inside the isolated guest.

### Owner-supplied Guard and proxy evidence

- `claryel-guard.service`: active, exited, enabled.
- `claryel-proxy-firewall.service`: active, exited, enabled.
- `inet claryel_guard` table is present.
- In `guest_input`, traffic arriving on `virbr240` is accepted only for established/related connections and DHCP replies (UDP source 68 to destination 67), then dropped. The displayed rules do not allow a new connection from the Cloud Hermes guest to the host proxy.
- In `guest_forward`, traffic entering or leaving `virbr240` is dropped.
- A Squid process is listening on `172.30.250.1:3128`.
- The displayed `guest_input` rule allows a new/established TCP connection to `172.30.250.1:3128` only when it arrives on `br-strata-eg` from source `172.30.250.10`.
- The `virbr240` guest is on `172.30.240.0/24`, while the Squid listener and permitted source are on `172.30.250.0/24`.

### Assessment

- **CONFIRMED:** The Cloud Hermes libvirt network definition has no forwarding element and is therefore configured as an isolated network.
- **CONFIRMED:** The guest has no default route (from the prior guest snapshot).
- **CONFIRMED:** The host Guard rules drop forwarding to/from `virbr240`.
- **CONFIRMED:** A Squid listener exists on `172.30.250.1:3128`.
- **CONFIRMED:** The displayed Guard allowance for that proxy is narrowly scoped to source `172.30.250.10` arriving on `br-strata-eg`; it does not include the Cloud Hermes guest or `virbr240`.
- **LIKELY:** The immediate PyPI failure is consistent with the intended isolated-network posture and the absence of an approved Cloud Hermes egress path, rather than a resolver daemon failure.
- **NOT_YET_DETERMINED:** Whether Squid is the intended Security Gateway for Cloud Hermes, whether a separate broker/connector exists, the Squid ACL/upstream policy, and the precise approved route from the Cloud Hermes zone to the independent gateway.
- **NO_CHANGES:** No libvirt network, bridge, route, nftables rule, proxy configuration, service, guest setting, or credential was changed.

### Next action

Continue with read-only host inspection of the proxy/firewall implementation:
1. Inspect the exact systemd unit and script for `claryel-proxy-firewall.service` and identify the rules it installs.
2. Inspect Squid ACL/listener/upstream policy without printing credentials or secret-bearing directives.
3. Identify the container/interface at `172.30.250.10` and the network it belongs to.
4. Check whether the existing policy/architecture documentation already defines a separate Security Gateway path for Cloud Hermes.

Do not attach `virbr240` to the Strata egress bridge, add NAT/default routing, permit the guest to Squid, enable generic internet forwarding, or weaken Guard until the intended broker design and policy are verified and the owner approves a specific change.

### Contribution provenance

- Writer-instance: CHATGPT-INSTALL-01 (declared; unauthenticated)
- Writer-role: INSTALLATION_COORDINATOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: Owner on P2002 host
- Evidence-source: OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-07 (read-only host diagnostics)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: CLARYEL-P2002-20261107-HOST-NETWORK-PROXY-TOPOLOGY-001

## Architecture and accepted Squid policy cross-check — PyPI is not an approved Strata proxy destination

- **Checkpoint ID:** CLARYEL-P2002-20261107-EGRESS-POLICY-CROSSCHECK-001
- **Target-machine-ID:** P2002
- **Deployment-alias:** P2002-LAB
- **Event time:** 2026-11-07 (repository review; exact time not material)
- **Operation:** READ_ONLY_REPOSITORY_POLICY_REVIEW
- **Evidence sources:** TOOL_OBSERVED_REPOSITORY_FILES + OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** ChatGPT reviewed repository documents; owner supplied host and guest terminal evidence. ChatGPT did not execute on P2002.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (continue installation diagnosis and repository review)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoints:** CLARYEL-P2002-20261107-HOST-NETWORK-PROXY-TOPOLOGY-001; CLARYEL-P2002-20261107-PM-REPAIR-DNS-FAILURE-001

### Existing architecture statements

Repository `docs/ARCHITECTURE.md` describes the public Hermes VM as having restricted external access and identifies an independently secured Security GTW as the sole typed cross-zone gateway. It explicitly says this architecture document is an original proposal and that the newer v0.2 document supersedes it.

Repository `docs/ARCHITECTURE-V0.2-REPLACEABLE-PLATFORM.md` describes a target architecture and explicitly states it is a proposal, not deployed capability or permission to change the live P2002 installation. It requires default-deny network enforcement and an independent Security Gateway for approved cross-zone requests.

### Existing Squid acceptance policy

Repository `logs/P2002/2026-11-06-strata-network-isolation-acceptance.md` records the already-tested Strata egress design:
- Docker network: `claryel-strata-egress`, subnet `172.30.250.0/24`.
- Squid endpoint: `172.30.250.1:3128`.
- Only authorized client: `172.30.250.10`.
- Squid permits HTTPS CONNECT only to `huggingface.co` and `us.aws.cdn.hf.co` for that client.
- Other requests are denied; GitHub was explicitly tested and denied.
- Direct container egress is denied.
- The tests were performed for the Strata egress client and did not validate Cloud Hermes guest access.

The current owner-supplied host snapshot matches that policy: the proxy listener is on `172.30.250.1:3128`, and Guard permits it only from `172.30.250.10` on `br-strata-eg`. The Cloud Hermes guest at `172.30.240.12` is not an authorized client.

### Assessment and decision

- **CONFIRMED:** The existing Squid service is a narrowly scoped Strata/Hugging Face egress proxy, not a general-purpose internet proxy.
- **CONFIRMED:** PyPI is not in the documented Squid allowlist; GitHub was explicitly denied.
- **CONFIRMED:** The Cloud Hermes guest is not included in the existing Squid proxy exception.
- **NOT_ESTABLISHED:** A deployed independent Security GTW or a policy-approved Cloud Hermes egress path. The architecture docs define the target role, but they do not prove it is implemented on P2002.
- **DECISION:** Do not add PyPI, Grok API, or general domains to the Strata Squid allowlist; do not attach Cloud Hermes to the Strata egress bridge; do not add NAT/default internet routing to `virbr240`. Those changes would weaken or repurpose a tested boundary without an approved Security GTW design.
- **INSTALLATION CONSEQUENCE:** Dependency repair remains blocked until a policy-approved package-intake path or a verified offline dependency cache is available. Hermes update and Gateway service installation remain blocked.

### Next action

Return to the existing guest and inspect the local PM dependency/cache state without network access:
1. Run the read-only `hermes pm doctor` (without `--repair`).
2. Inspect the uv cache directory size and whether cached artifacts for `setuptools==84.1.0` and `wheel` are available.
3. Inspect the existing PM install/environment state without reading credential files or printing environment variables.

If the required artifacts are already cached, evaluate a bounded offline repair. If not, record the missing artifacts as a package-intake task and design a narrow, verified quarantine transfer; do not open general egress.

### Contribution provenance

- Writer-instance: CHATGPT-INSTALL-01 (declared; unauthenticated)
- Writer-role: INSTALLATION_COORDINATOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: ChatGPT for repository review; owner for P2002 host/guest terminal evidence
- Evidence-source: TOOL_OBSERVED_REPOSITORY_FILES + OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-07 (continue installation diagnosis and repository review)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: CLARYEL-P2002-20261107-EGRESS-POLICY-CROSSCHECK-001

## Owner-supplied Hermes cache inspection — required build packages are present

- **Checkpoint ID:** CLARYEL-P2002-20261107-PM-CACHE-SETUPTOOLS-WHEEL-PRESENT-001
- **Target-machine-ID:** P2002-CLOUD-GUEST
- **Deployment-alias:** P2002-LAB
- **Event time:** UNKNOWN (no timestamp in supplied terminal transcript)
- **Operation:** READ_ONLY_PM_CACHE_AND_RECOVERY_STATE_INSPECTION
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner, as the existing non-root guest user in the already-open VM shell; ChatGPT did not execute on the guest.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (read-only cache inspection)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoints:** CLARYEL-P2002-20261107-PM-REPAIR-DNS-FAILURE-001; CLARYEL-P2002-20261107-GUEST-NETWORK-NO-DEFAULT-ROUTE-001

### Owner-supplied observations

- Hermes-managed UV cache at `/home/claryelvm/.hermes/cache/uv`: **309 MB**.
- Cached unpacked package metadata directories:
  - `setuptools-84.1.0.dist-info`
  - `wheel-0.48.0.dist-info`
  - `wheel-0.46.3.dist-info`
- Hermes cache seed marker exists and contains digest `5057a8a3cbc6d2b1f2059d66e27ffa43f447021f89779599dca416056574c064`.
- Repair marker at `~/.hermes/installs/ab16f6e41f23151a/.repair-incomplete` contains `{"attempts": 1}`.
- Two existing PM runtime generations are present:
  - `518cf5d398374517a65e662881e8eeae`
  - `21a9bd4435d448a19d8ddc553e3a9c14`

### Exact-version source review (Hermes v0.23.0, commit 2bd378ed1b8d6bf2192cb26e0a55ff3091e6bd19)

- `pm.packages.uv_cache_path()` resolves the Hermes-managed cache to `<HERMES_HOME>/cache/uv`; the earlier `~/.cache/uv` check was not the PM cache.
- `pm.environment.PythonEnvironment._run()` explicitly sets `UV_CACHE_DIR` to the PM cache and `UV_PYTHON_DOWNLOADS=never`.
- `pm.environment._base_environment()` strips ambient UV settings except those allowed by `pm.index_config.is_forwarded()`.
- `pm.index_config.FORWARDED_UV_SETTINGS` includes `UV_NO_INDEX`, but does **not** include `UV_OFFLINE`. Therefore setting `UV_OFFLINE=1` in the shell would not be a reliable way to force Hermes PM's child uv process offline.
- `pm.cli.cmd_repair()` has no offline command-line option; it restores the recorded graph via `recover_if_needed(..., explicit=True)`.
- The PM repair path invokes `sync_venv(repair=True)`, which builds a fresh dependency generation and retains the recorded dependency graph.

### Assessment

- **CONFIRMED:** The correct Hermes-managed cache is populated (309 MB), unlike the unrelated default `~/.cache/uv` directory (32 KB).
- **CONFIRMED:** Cache metadata for the exact setuptools pin from the failed repair (`84.1.0`) and for wheel distributions is present.
- **NOT_YET_CONFIRMED:** The cache contains every required wheel/archive and build dependency in a form sufficient for the complete locked repair. Presence of `.dist-info` metadata alone does not prove all artifacts are reusable.
- **CONFIRMED:** The incomplete repair marker records one failed attempt.
- **CONFIRMED:** Existing runtime generations remain present; no evidence in this transcript indicates they were deleted.
- **NO_CHANGES:** This cache inspection was read-only.

### Next controlled action

Attempt the recorded PM repair while explicitly disabling package-index lookup using `UV_NO_INDEX=1`:
`UV_NO_INDEX=1 "$HOME/.local/bin/hermes" pm repair`

Exact-version source confirms `UV_NO_INDEX` is forwarded into the child uv environment, unlike `UV_OFFLINE`. The guest also has no default route and host Guard currently blocks forwarding for `virbr240`, so no ordinary network route to PyPI is available. This attempt must be treated as a bounded cache-only test: if any required artifacts are absent, expect a clear failure and stop; do not add a package index, route, DNS server, or proxy exception. Do not retry repeatedly.

On success, verify PM doctor and the repair marker before proceeding to Hermes update or Gateway installation. Do not run those later steps until the repaired generation is confirmed.

### Contribution provenance

- Writer-instance: CHATGPT-INSTALL-01 (declared; unauthenticated)
- Writer-role: INSTALLATION_COORDINATOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: Owner in the existing guest shell for terminal evidence; ChatGPT for exact-version source review
- Evidence-source: OWNER_SUPPLIED_TERMINAL_TRANSCRIPT + TOOL_OBSERVED_HERMES_SOURCE
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-07 (read-only cache inspection; next state change requires owner to run the displayed command)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: CLARYEL-P2002-20261107-PM-CACHE-SETUPTOOLS-WHEEL-PRESENT-001

## Bounded cache-only repair attempt failed — correction to UV_NO_INDEX assumption

- **Checkpoint ID:** CLARYEL-P2002-20261107-PM-UV-NO-INDEX-REPAIR-FAILED-001
- **Target-machine-ID:** P2002-CLOUD-GUEST
- **Deployment-alias:** P2002-LAB
- **Event time:** UNKNOWN (no timestamp in supplied terminal transcript)
- **Operation:** OWNER_EXECUTED_PM_REPAIR_WITH_UV_NO_INDEX_ENV
- **Evidence source:** OWNER_SUPPLIED_TERMINAL_TRANSCRIPT
- **Executed-by:** Owner, as the existing non-root guest user in the already-open VM shell; ChatGPT did not execute on the guest.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (single bounded repair attempt)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoints:** CLARYEL-P2002-20261107-PM-CACHE-SETUPTOOLS-WHEEL-PRESENT-001; CLARYEL-P2002-20261107-PM-REPAIR-DNS-FAILURE-001

### Owner-supplied command and result

Command:
`UV_NO_INDEX=1 "$HOME/.local/bin/hermes" pm repair`

Observed:
- PM repair began and attempted to build a fresh generation under `~/.hermes/installs/ab16f6e41f23151a/environments/6caee0fc9a5944b0ac46b48ff5dce014/workspace`.
- uv still attempted `https://pypi.org/simple/setuptools/`.
- It failed after three retries with DNS resolution failure; uv sync exited 2.
- No successful dependency repair is established.
- No evidence of a successful Gateway service installation/start is established.

### Correction to prior analysis

The previous checkpoint incorrectly described `UV_NO_INDEX=1` as a reliable cache-only control because Hermes v0.23.0's `pm.index_config.FORWARDED_UV_SETTINGS` includes that name. The owner-supplied result disproves that operational assumption: the actual uv process still queried PyPI.

Further review of uv's official environment-variable reference shows **no documented `UV_NO_INDEX` environment variable**. The uv CLI does support the command-line flag `--no-index`, but Hermes's `hermes pm repair` command does not expose that flag. uv documents `UV_OFFLINE` as its supported environment variable for disabling network access; however, Hermes v0.23.0's `pm.environment._base_environment()` strips ambient `UV_*` variables unless listed in `FORWARDED_UV_SETTINGS`, and `UV_OFFLINE` is not on that allowlist. Therefore neither prior shell-variable attempt constitutes a proven offline repair method.

Official uv source: https://docs.astral.sh/uv/reference/environment/ (the reference documents `UV_OFFLINE`; it has no `UV_NO_INDEX` entry).
Exact Hermes source reviewed at commit `2bd378ed1b8d6bf2192cb26e0a55ff3091e6bd19`:
- `pm/index_config.py`: forwarding allowlist
- `pm/environment.py`: ambient UV filtering and child environment construction
- `pm/client.py` and `pm/runtime.py`: isolated PM worker environment
- `pm/install.py`, `pm/packages.py`, and `pm/workspace.py`: repair builds a fresh environment generation and replays the recorded workspace/lock

### Assessment

- **CONFIRMED:** The attempted repair still requires resolving build-system requirements from PyPI; the guest cannot resolve/reach PyPI.
- **CONFIRMED:** The prior claim that `UV_NO_INDEX=1` would make Hermes repair cache-only was incorrect and is withdrawn.
- **NOT_YET_CONFIRMED:** Whether the cache contains reusable wheel artifacts and index metadata sufficient to satisfy the build-system requirements without a registry.
- **NOT_YET_CONFIRMED:** Whether the failed candidate generation was removed as expected; verify read-only before further action.
- **NO_NETWORK_POLICY_CHANGE:** No guest route, DNS, NAT, forwarding, Guard rule, Squid ACL, or proxy exception was changed.
- **NO_GATEWAY_INSTALL:** Gateway install remains blocked pending successful dependency recovery.

### Next controlled action — READ ONLY

Inspect the exact UV cache layout and package artifacts (not only `.dist-info` directories), the installed PM uv executable/version, the repair marker, and current environment generation directories. Do not run another repair, do not set further guessed UV environment variables, and do not open guest egress.

If wheel archives are missing, plan a narrow package-intake transfer with verified hashes through an approved host-to-guest path. If artifacts exist but uv still lacks registry metadata, evaluate a local file-based package index / wheelhouse using a supported uv mechanism; validate this design before attempting any mutation. Do not change the upstream Hermes checkout or lockfile ad hoc.

### Contribution provenance

- Writer-instance: CHATGPT-INSTALL-01 (declared; unauthenticated)
- Writer-role: INSTALLATION_COORDINATOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: Owner for terminal command; ChatGPT for source and official uv documentation review
- Evidence-source: OWNER_SUPPLIED_TERMINAL_TRANSCRIPT + TOOL_OBSERVED_HERMES_SOURCE + OFFICIAL_UV_DOCUMENTATION
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-07 (bounded repair attempt and follow-up diagnosis)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: CLARYEL-P2002-20261107-PM-UV-NO-INDEX-REPAIR-FAILED-001

## Owner-supplied UV cache inspection — cached package trees and index metadata verified

- **Checkpoint ID:** CLARYEL-P2002-20261107-PM-CACHE-ARCHIVE-AND-INDEX-INSPECTED-001
- **Target-machine-ID:** P2002-CLOUD-GUEST
- **Deployment-alias:** P2002-LAB
- **Event time:** UNKNOWN (the owner supplied no timestamp with this transcript)
- **Operation:** READ_ONLY_UV_CACHE_STRUCTURE_AND_DISTRIBUTION_INTEGRITY_CHECK
- **Evidence source:** OWNER_REPORTED terminal transcript; scope is the existing Cloud Hermes guest only.
- **Executed-by:** Owner in the already-open guest shell. ChatGPT did not execute commands on the guest.
- **Prepared-by:** ChatGPT GPT-6 (declared)
- **Writer-instance:** CHATGPT-INSTALL-01 (declared; unauthenticated)
- **Writer-role:** INSTALLATION_COORDINATOR
- **Hermes-runtime-machine-ID:** NOT_APPLICABLE (the writer is ChatGPT, not a Hermes instance)
- **Deployment-alias:** P2002-LAB
- **Reviewed-by:** SELF_REVIEW_ONLY
- **Approved-by:** OWNER_REQUEST_2026-11-07 (continue the owner-controlled installation workflow and publish its audit evidence)
- **Published-by:** GitHub connector via shared ViktorIXAI account
- **Attribution-confidence:** DECLARED_UNVERIFIED
- **Related-issue:** #10
- **Related-checkpoint:** CLARYEL-P2002-20261107-PM-CACHE-SETUPTOOLS-WHEEL-PRESENT-001

### Observed cache structure

The owner inspected the Hermes-managed UV cache. Results:
- `simple-v27` contains:
  - `pypi/setuptools.rkyv` (400,100 bytes)
  - `pypi/packaging.rkyv` (31,092 bytes)
  - `pypi/wheel.rkyv` (44,686 bytes)
- The wheel cache listing contains a setuptools entry:
  - `pypi/setuptools/84.1.0-py3-none-any.msgpack` (2,668 bytes)
  - corresponding `.http` metadata (548 bytes) and `.lock`.
- The cache contains many other package HTTP metadata entries and lock files.
- The only `.whl` archive returned by the owner's bounded search was the editable Hermes project wheel. No setuptools or wheel distribution wheel archive was found by that search.
- The source-distribution cache listing contains editable-project metadata and the editable Hermes project wheel; it did not show a setuptools or wheel sdist archive.
- The attempted `rg` command was unavailable in that interactive shell, despite the earlier PM doctor reporting ripgrep as present. This discrepancy is not yet explained and is not currently on the critical installation path.

### Distribution tree integrity verification

The owner ran a Python read-only verifier against the unpacked UV cache entries:
- `setuptools 84.1.0`: 343 files checked against its `RECORD`; no missing files and no hash mismatches.
- `wheel 0.48.0`: 20 files checked against its `RECORD`; no missing files and no hash mismatches.
- Both contain `WHEEL` metadata identifying pure-Python `py3-none-any` distributions.
- The earlier failed candidate generation from the prior repair attempt was absent.
- One existing PM environment generation remained in the inspected directory.
- The repair marker still records two attempts.

### Assessment

- **CONFIRMED:** The UV simple-index cache has metadata entries for setuptools and wheel.
- **CONFIRMED:** The unpacked cached distribution trees for setuptools 84.1.0 and wheel 0.48.0 pass their own RECORD file/hash verification.
- **CONFIRMED:** The bounded `.whl` search did not find distributable wheel archives for those two packages.
- **NOT_YET_CONFIRMED:** Whether the cached index records and HTTP metadata are fresh/complete enough for uv to resolve without contacting the index.
- **NOT_YET_CONFIRMED:** Whether a standards-compliant wheel can be safely reconstructed from the unpacked cached trees using an already-installed wheel utility, without modifying the active PM environment.
- **NOT_YET_CONFIRMED:** Whether the remaining PM generation is healthy and contains a usable wheel-packing utility.
- **NO_CHANGES:** This inspection was read-only. No cache, PM environment, lockfile, network policy, service, or Hermes configuration was changed.

### Next controlled action — read-only capability check

Inspect the existing retained PM environment only:
1. Identify the interpreter in the retained generation's `venv` and check whether the `wheel` package/module and `wheel pack` command are already installed.
2. Check the same for the PM-managed uv executable/version and its supported local-source options.
3. Inspect only package names/versions and executable paths; do not dump environment variables, credentials, user configuration, or secret-bearing files.

If an already-installed wheel packer is available, evaluate reconstructing verified local wheel artifacts from the intact extracted distribution trees into a separate temporary wheelhouse, without changing the retained generation. Before any such write, present the exact proposed command and output path for owner approval. Do not run another PM repair, modify the lockfile, change Hermes source, or open guest egress.

### Contribution provenance

- Writer-instance: CHATGPT-INSTALL-01 (declared; unauthenticated)
- Writer-role: INSTALLATION_COORDINATOR
- Deployment-alias: P2002-LAB
- Prepared-by: ChatGPT GPT-6 (declared)
- Executed-by: Owner for guest terminal inspection; ChatGPT for interpretation and documentation
- Evidence-source: OWNER_REPORTED_TERMINAL_TRANSCRIPT
- Reviewed-by: SELF_REVIEW_ONLY
- Approved-by: OWNER_REQUEST_2026-11-07 (continue installation workflow; next state-changing command remains owner-approved separately)
- Published-by: GitHub connector via shared ViktorIXAI account
- Attribution-confidence: DECLARED_UNVERIFIED
- Related-checkpoint: CLARYEL-P2002-20261107-PM-CACHE-ARCHIVE-AND-INDEX-INSPECTED-001
