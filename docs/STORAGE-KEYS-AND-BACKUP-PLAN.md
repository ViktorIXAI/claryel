# P2002 Storage, Secrets, Lock States and Deferred Backup Design

**Status: owner-approved design targets, NOT deployed or security-tested. Updated 2026-11-06.** All numbers are approximate GB; distinguish manufacturer decimal GB from GiB shown by Linux. Validate physical NVMe size, current partition table and recoverability before any destructive change.

## First installation

- Allocate approximately **350 GB** to the LUKS2-encrypted Ubuntu Server 24.04 system environment (including OS, KVM, Docker/Compose, cloud VM, local Strata and models, Hermes agents, GTW, Home Assistant and operational metadata). Aim to reserve **100 GB** of this capacity for model weights and Strata assets; the current main model occupies about **78 GB** per owner report, so this is a capacity *reservation*, not a separate partition or a guarantee for future models.
- Plan a **32 GB encrypted swapfile** inside the encrypted system filesystem. Swap must never be configured unencrypted; disabling hibernation and monitoring memory pressure are part of acceptance.
- Allocate **16 GB** to an independently passphrase-unlocked LUKS2 secrets volume, to be created after the OS installation.
- Allocate the remaining capacity to a separate independently passphrase-unlocked LUKS2 private-data volume (documents, sensitive databases, user files and private RAG). For the initial install, leave this capacity unallocated rather than risking complex manual partitioning.
- Clarify physical meaning: these are **logical encrypted volumes on a single physical NVMe** unless a distinct physical drive is inventoried. Separate volumes/keys protect access, but do not protect against loss of the physical disk or host compromise.
- Boot partitions may be unencrypted; document Secure Boot and boot integrity limitations. LUKS system unlock may require a local passphrase at boot, affecting unattended restart. Never store private/secrets volume passphrases or raw unlock keys on the system disk or in a public repo.

## Storage budget inside 350 GB (planning caps, not measured use)

| Category | Planned budget |
| --- | ---: |
| System packages, kernels and base tools | 25 GB |
| Encrypted swap | 32 GB |
| Strata + model weights and model artifacts | **100 GB** |
| Docker images, containers and writable layers | 45 GB |
| KVM cloud VM and images | 45 GB |
| Hermes, Home Assistant, GTW auxiliary state and application binaries | 18 GB |
| Logs, update quarantine, temporary working space | 30 GB |
| Free growth/headroom | **55 GB** |
| **Total** | **350 GB** |

Budgets overlap in implementation (Docker layers may contain application files); use them only as capacity ceilings/monitoring categories, not an automatic quota map. In particular keep private customer data and secrets out of container writable layers, VM snapshots, model files, rootfs logs and backup artifacts. Do not rely on the initial 100 GB model allocation for arbitrarily larger models. Use disk alerts, safe container log rotation and measured size reports.

## Data-state indicator and control policy

Every relevant CLARYEL application/dashboard must show a small consistent icon with explicit data state and scope, rather than relying on color alone:

- 🔒 **LOCKED**: private-data volume not mounted, no authorized path to plaintext through active application services; verify state rather than merely displaying intended state.
- 🔓 **UNLOCKED — INTERNAL ACCESS ONLY**: private-data volume is mounted; only authenticated/authorized internal identities may access through appropriately encrypted transport or disk storage. Unlocked does **not** imply publicly accessible.
- ⏳ **TRANSITIONING / UNKNOWN**: deny access, show warning and do not claim locked until teardown is verified.
- ⛔ **LOCK FAILED**: access cannot be proven revoked; fail closed on new requests, alert an administrator, possibly power off to protect residual memory.

An app GUI icon is **not** itself cryptographic enforcement. State comes from a restricted OS-owned controller verifying mount, LUKS and application handles. A lock action must halt new requests, drain/stop private workloads, clear agent/model contexts where supported, prevent writes, unmount and close LUKS only after verification; a forced lockdown can require shutting down/private-process termination or power-off. CPU/GPU memory, swap, caches and already transmitted data need separate threat treatment; no promise of instantaneous erasure. The public Hermes may continue operating with no private data access.

## Initial secrets and licensing design

The secrets volume is initially passphrase-controlled and manually unlocked. Keys/passphrases are separate from private-data and system keys. **Secrets Broker** is a future independently restricted service with narrowly scoped identities, policy-based issuance, audit without secret values, no arbitrary agent administrative access, and offline-capable operation. Do not enable agent access to a master vault or run a privileged secrets API without verification.

Evaluate two different types of components without equating them:
- **KeePassXC / KDBX**: candidate for human-operated offline password database; GPL-3.0 requirements and redistribution/packaging obligations must be reviewed before inclusion in a commercial product (see official documentation). Not inherently a machine-to-machine secrets broker.
- **OpenBao**: candidate dedicated API-based secrets manager for service credentials and dynamic secrets; MPL-2.0, but much greater security and operational complexity, including authentication, sealed-state management and policy governance. Review actual current license, dependency licenses, threat boundaries and EU legal obligations before product bundling.

Do not automatically install either service in the first storage provisioning step. Initial secrets can be manually managed on an isolated encrypted volume while a narrowly privileged Secrets Broker is designed and tested. An encrypted vault protects secrets *at rest*, not while applications use them in memory.

## Backups: design recorded, owner has NOT reviewed or approved implementation

**Explicit owner status: DEFERRED / NOT YET OWNER-REVIEWED / NOT TESTED.** Create no automatic backup service yet. Document intent only:
- System configuration and reproducible manifests: at changes and periodic versioned snapshots;
- Encrypted private data: incremental and periodic full snapshots with retention/erasure policy;
- Secret vault and LUKS recovery/header material: tightly controlled independent encrypted offline recoverability;
- Independent copies on separate physical storage, restored offline in regular drills;
- No GitHub copy of private backup payloads or credentials, and no false claim that GitHub history backs up all application state.

Backups can increase exposure and conflict with data retention/deletion if unmanaged. The exact retention, frequency, destinations, encryption keys, access and recovery procedures remain **open for future owner review after the base system is deployed**.

## Immediate action boundary

Install Ubuntu on the intended 350 GB encrypted system area only; do **not** erase or reformat the remainder unintentionally. Never run partition/cryptsetup commands based on assumed disk names: first inspect actual `lsblk`, storage IDs and the installer preview. Later creation of independently encrypted secrets/data volumes requires explicit review of the actual geometry. No real customer files or production credentials before isolation tests.
