# P2002 Storage, Secrets, Lock States and Deferred Backup Design

**Status: owner-approved testbed design targets, NOT deployed or security-tested. Updated 2026-11-06.** All numbers are approximate GB; distinguish manufacturer decimal GB from GiB shown by Linux. Validate physical NVMe size, current partition table and recoverability before any destructive change.

## First installation

- Allocate approximately **350 GB** to an **UNENCRYPTED Ubuntu Server 24.04 system environment for the initial P2002 testbed only** (including OS, KVM, Docker/Compose, cloud VM, local Strata and models, Hermes agents, GTW, Home Assistant and operational metadata). Aim to reserve **100 GB** of this capacity for model weights and Strata assets; the current main model occupies about **78 GB** per owner report, so this is a capacity *reservation*, not a separate partition or a guarantee for future models.
- Plan **32 GB swap**, but on the initial unencrypted system do not place actual private data, real client documents, secret keys, sensitive contexts or plaintext credentials in memory/swap. Prefer an explicitly encrypted ephemeral swap mechanism when feasible, or disable swap until it can be protected; verify after each restart. No hibernation with ephemeral swap. This is a temporary test-only security exception.
- Allocate **16 GB** to an independently passphrase-unlocked LUKS2 secrets volume, to be created after the OS installation.
- Allocate the remaining capacity to a separate independently passphrase-unlocked LUKS2 private-data volume (documents, sensitive databases, user files and private RAG). For the initial install, leave this capacity unallocated rather than risking complex manual partitioning.
- Clarify physical meaning: these are **logical encrypted volumes on a single physical NVMe** unless a distinct physical drive is inventoried. Separate volumes/keys protect access, but do not protect against loss of the physical disk or host compromise.
- Boot partitions and the initial system partition are **unencrypted**. Document physical-access, offline tampering, log, Docker, VM, crash-dump and swap exposure risks. **No in-place root encryption will be attempted.** The owner selected a complete disk reinstallation with LUKS2 during an integrated backup, disaster recovery, and encryption exercise, using independently verified recovery data. Never store private/secrets volume passphrases or raw unlock keys on the system disk or in a public repo.

## Storage budget inside 350 GB (planning caps, not measured use)

| Category | Planned budget |
| --- | ---: |
| System packages, kernels and base tools | 25 GB |
| Swap (test-only; encryption/disable policy required) | 32 GB |
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

## Backups and disaster recovery: implementation deferred

**Explicit owner status: CONCEPT APPROVED; IMPLEMENTATION DETAILS NOT YET REVIEWED; NOT DEPLOYED OR TESTED.** The owner requires a future combined backup, destructive reinstall, encrypted restore and verification exercise. Create no automatic backup service yet. Document intent only:
- System configuration and reproducible manifests: at changes and periodic versioned snapshots;
- Encrypted private data: incremental and periodic full snapshots with retention/erasure policy;
- Secret vault and LUKS recovery/header material: tightly controlled independent encrypted offline recoverability;
- Independent copies on separate physical storage, restored offline in regular drills;
- No GitHub copy of private backup payloads or credentials, and no false claim that GitHub history backs up all application state.

Backups can increase exposure and conflict with data retention/deletion if unmanaged. The exact retention, frequency, destinations, encryption keys, access and recovery procedures remain **open for future owner review after the base system is deployed**.

## Immediate action boundary

Install Ubuntu on the intended 350 GB **unencrypted** system area only for this first testbed; do **not** erase or reformat the remainder unintentionally. Never run partition/cryptsetup commands based on assumed disk names: first inspect actual `lsblk`, storage IDs and the installer preview. Later creation of independently encrypted secrets/data volumes requires explicit review of the actual geometry. No real customer files or production credentials before isolation tests.

## Owner change decision (2026-11-06)

The project owner explicitly selected an **unencrypted 350 GB system root for the initial P2002 testbed**, to speed initial deployment. This supersedes the earlier encrypted-system recommendation **only for this test phase**. Independently encrypted secrets and private-data volumes remain mandatory before any sensitive material is introduced. **In-place conversion is explicitly excluded.** Later system encryption will be introduced **only through a destructive clean reinstall** of the test system, then restoration from separately stored and verified backup material, with an end-to-end rebootable recovery test. This future exercise will test backup, restore, LUKS2 root encryption, swap, secrets, logs, VM and container state **as one integrated workflow**. No claim that an unencrypted host disk is adequate for real customer data.

## Future integrated backup–wipe–reinstall–restore–encrypt exercise (owner decision, 2026-11-06)

- **Now:** deploy the P2002 testbed on the unencrypted 350 GB system volume using synthetic data only. Do not configure automatic backup jobs or encrypt the system root as part of initial deployment.
- **Later, as a separate explicitly approved test:** design and implement independent backup destinations, encryption/key custody, immutable/configuration and state exports, VM/container/database-consistent backup methods, retention and restoration procedures. This design is currently unverified.
- **Mandatory pre-wipe gates:** complete at least one independent trial restore on a separate test location; verify image/manifests/database and secret-store coverage, file integrity and recoverability; confirm independent copies exist off the target disk; inventory ALL disks and relevant partitions; confirm recovery keys are available independently; record owner authorization specifically for irreversible erasure. A backup report alone is insufficient.
- **Destructive exercise (not authorized yet):** deliberately wipe only the owner-approved target system after prerequisites pass; reinstall a fresh Ubuntu system with LUKS2-encrypted root and protected swap; restore services, configs, models and synthetic test data from verified backups; re-establish separate protected secrets/private-data volumes and boundaries. **Never perform an in-place cryptsetup re-encryption of the existing root.**
- **Acceptance:** boot/reboot with LUKS, verify locked/unlocked indicators, recover SSH and administration, check service isolation and offline operation, test KVM/Docker/Hermes/Strata/GTW/HA, integrity/permissions, restore speed and data completeness, encrypted-at-rest state and rollback/failure drills. Publish only sanitized evidence.
- If any recovery gate fails, **STOP without erasing anything**. No customer data or production credentials are allowed in the unencrypted test environment.
