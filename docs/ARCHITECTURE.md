# Architecture v0.1 — proposal, not deployment authorization

## Isolation zones

| Zone | Internet | Private documents | Role |
| --- | --- | --- | --- |
| Hardened Ubuntu host / hypervisor | Controlled maintenance only | Encrypted storage, privileged access controlled | Isolation base |
| Private Hermes container | None | Scoped by authenticated user and policy | Internal tasks |
| Strata + local LLM | None | No direct mounts; only authorized prompt data | Inference-only service |
| Public Hermes VM | Restricted external access | Never, under strict policy | Web/cloud integrations |
| Security GTW | No general-purpose Internet | No direct data-store mount | Sole typed cross-agent gateway |
| Home Assistant | Denied by default; reviewed device exceptions | None by default | Local devices |
| Quarantine/staging | Controlled download path | None | Untrusted updates |
| Backup/admin | Separate limited privileges | Authorized, encrypted and audited | Recovery and operation |

## Mandatory interaction boundary

Public Hermes **must never** call Private Hermes directly, and vice versa. All cross-zone exchanges traverse an independently secured **Security GTW**, with narrow typed operations (e.g. PUBLIC_WEB_SEARCH), identities, authorization, payload validation and rate/size limits. No transparent general-purpose proxy, shared files, IPC sockets or agent-managed GTW rules. The protected side initiates approved queries; the external agent may not initiate unsolicited operations inside. Untrusted external output is data, never instructions or authorization.

First-stage strict mode forbids transmitting private documents, retrieved snippets, embeddings, identifying details, confidential context or raw private logs to the external zone. Content filtering is defense-in-depth, **not** a sufficient guarantee: prevent unauthorized information from entering external channels upstream.

## Local inference sharing

The owner's initial hypothesis uses a single physical GPU and possibly a single Strata inference process for both agents, accessed only through separated API gateways. A shared process must be treated as an **unverified isolation assumption**: independently inspect session lifecycle, prompt/KV/prefix caches, slots, concurrent batching, logging, tracing, error reporting and any plugin/debug endpoints. Design for later separation into distinct processes/containers or VMs without changing GTW contracts. Every external response must be buffered and policy-checked before release; never stream unreviewed tokens to cloud.

Strata receives only scoped inference requests; deny arbitrary file access, MCP/tools, shell, network, admin endpoints, host sockets and unapproved model loading. Enforce isolation in host firewall, kernel permissions, container runtime and API authorization (including IPv6 and Docker forwarding). Any tool-looking model output is untrusted text until separately authorized.

## Offline and recovery

Disconnect WAN physically in acceptance tests. Private inference, data access, local identity and supported HA automations must work offline. Updates, cloud search and cloud-only integrations must fail explicitly without opening permissive fallback routes. Backups are encrypted, independently restorable and tested; keys and logs stay outside the public repository.

## Threats to test

Prompt injection, compromised agents, arbitrary model output, supply-chain insertion, malicious HA devices, cross-tenant contamination, shared-cache leaks, SSRF/API smuggling, network/IPv6/DNS bypass, privilege escalation, host/VM/container escape, secret logging, physical loss and failed updates. A hostile host or hypervisor is beyond guarantees possible solely through containers; document residual risks.
