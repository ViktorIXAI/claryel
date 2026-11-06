# Initial Hardware Testbeds and Evaluation Roles

**Status: owner-provided planning baseline; hardware specifications and prices are approximate and must be validated before benchmarking. No installations or benchmark results are claimed here.**

CLARYEL uses two distinct physical machines with complementary purposes. **The full system is initially deployed and developed on the more capable P2002. P1005 is not intended to be an automatically maintained duplicate of the complete installation.** Instead, P1005 will test whether selected components, workloads and practical use cases perform adequately on less expensive hardware.

| Host | Owner description | Approximate system cost | Planned purpose |
| --- | --- | ---: | --- |
| **P2002** | ASUS ATX, 64 GB RAM, AMD GPU with 16 GB VRAM | **€2,500** | Primary architecture deployment, system configuration, integration, security controls and end-to-end verification |
| **P1005** | ASUS server, 32 GB DDR4 RAM, Tesla 16 GB GPU | **€1,000** | Budget hardware performance, feasibility and real-world task testing; selectively reproduce only what each experiment requires |

**GPU model verification:** The latest owner message refers to a “Tesla T100 16GB”; earlier equipment notes identify P1005 as equipped with a **Tesla P100 16GB**. This discrepancy must be resolved against a local hardware inventory (e.g., `nvidia-smi`) before specifying drivers, runtimes or benchmark comparisons. No automatic correction of owner-reported facts is assumed.

## Evaluation design

1. **P2002 is the reference development host.** Reproduce the clean-room staged architecture and document configuration, validation, GTW behavior, offline operation and security limitations. No production or client data is used before approved gates.
2. **P1005 is a budget feasibility host.** Validate target workloads and only the subset of services needed for the measurement. It is not a live redundant node, mirrored production system, or a substitute for P2002's full security integration tests.
3. **Comparable workloads, not unsupported comparisons.** Specify model hash and quantization, runtime and driver versions, context length, concurrent users, prompt lengths, output limits and hardware power constraints. Identify differences and partial functionality explicitly.
4. **Measure practical usability.** Report prefill latency, time to first token, sustained tokens/sec, end-to-end agent task latency, correctness/task success, memory/VRAM use, power draw, idle consumption, thermals, restart behavior, offline functionality and failure rates.
5. **Measure costs without invented precision.** €2,500 and €1,000 are approximate owner-estimated acquisition/system values, not verified current market prices. Record electricity price assumptions, measured watts, workload duration, depreciation where relevant, and total cost per usable task.
6. **Include negative evidence.** Publish reproducible failed trials, limits, incompatibilities and anomalies along with successes. Never imply that isolated model benchmark speed proves real-world task suitability or system security.
7. **Protect privacy and security.** Use only synthetic inputs and sanitized output/logs in public reports. Neither test machine is implicitly trusted; updates and deployments remain gated by the established project policies.

## Acceptance milestones

- Verify hardware inventory and especially the P1005 Tesla GPU name.
- Establish a functioning, documented baseline on P2002 under the owner-controlled roadmap.
- Define a repeatable workload suite from real-world task categories using synthetic data.
- Run the relevant subset on P1005; document unsupported features instead of silently altering the test contract.
- Publish a two-system comparison with measured performance, costs, accuracy, operational constraints and remaining uncertainty.

**Decision criterion:** whether the lower-cost P1005 delivers adequate performance for specific practical workloads, without pretending it replicates the complete P2002 installation.
