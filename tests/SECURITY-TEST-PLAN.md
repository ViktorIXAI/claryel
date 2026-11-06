# Security acceptance tests — synthetic data only

Use invented customer profiles, uniquely tagged canary secrets and test-only credentials. Never test with real client data or upload unredacted evidence.

- **WAN disconnected:** private Hermes, Strata, private document workflow and supported HA automations function; cloud features fail closed without route changes.
- **Network isolation:** no direct Hermes-to-Hermes network channel; external VM cannot reach private LAN/Docker/host services; private agent/model cannot reach WAN via IPv4, IPv6, DNS, DoH or unexpected protocols. Repeat after reboot and container changes.
- **GTW:** only typed authenticated allowlisted requests accepted; block file contents, embeddings, personal data, oversized payloads, replay, forged identities, injection and untrusted responses attempting local commands. Log decisions without payload secrets.
- **Model isolation:** test private canaries vs public requests under concurrency, KV/prefix caching, reused sessions, restarts and crashes; inspect trace/log leakage and API/admin endpoints. Any observed leak blocks promotion. A clean test does not prove absence of leaks.
- **Privilege:** attempt unauthorized filesystem, socket, Docker, tools/MCP, SSRF, model-loading and shell calls from test boundaries.
- **Updates:** reject tampered digests, new privileged mounts, changed firewall rules, unexpected telemetry; validate rollback and offline install.
- **Recovery:** restore independent encrypted backups without internet; validate permissions, integrity, data deletion and loss scenarios.
- **Legal/process:** record controller/processor responsibilities, processing inventory, DPIA decision, incident simulations, retention and user-right workflows.

For each test preserve versioned preconditions, exact steps, sanitized outcomes, evidence hashes, residual risks and owner sign-off. Never fabricate a pass.
