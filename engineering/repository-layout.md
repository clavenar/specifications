# Clavenar — Repository Layout

**Document revision:** 0.1  
**Date:** 24 November 2026  
**Organization:** `clavenar`  
**Primary repository:** `clavenar/core`  
**Website:** `clavenar.io`  
**Related document:** *Clavenar — Architecture and Vision*, draft 0.9  
**Status:** Proposed engineering baseline. Repository names, directories, crates and release artifacts below describe the intended organization, not a claim that they have already been created or implemented.

> One execution model, multiple deployment environments, a shared security contract.

## 1. Repository strategy

Clavenar is an agent-native operating system and execution platform with three deployment modes:

- **Clavenar Native / Bare Metal:** the independent Rust kernel and its native userland, with RV64 as the reference architecture and an early AMD64 port.
- **Clavenar Hosted:** the Linux-hosted execution runtime, using the same high-level goal, agent, capability and policy semantics but Linux enforcement mechanisms.
- **Clavenar Virtual:** the native kernel running as a VM guest, managed by host-side Goal Machine infrastructure. Virtual is a deployment of Native, not another operating system or kernel.

Use a public engineering monorepo, `clavenar/core`, for the shared implementation. Keep the public specifications and website separate; split other repositories only when independent ownership, release cadence, distribution or access control warrants it.

Native OS development remains the primary engineering objective. Hosted development accelerates tool and model integration, supplies the restricted QEMU bridge and provides a comparison baseline. Virtual introduces the native OS to existing Linux/KVM infrastructure without requiring customers to replace their host operating systems.

### 1.1 GitHub organization

| Repository | Visibility | When to create | Responsibility |
|---|---|---|---|
| `clavenar/core` | Public | Day one | Native kernel and userland, Hosted, Virtual, shared crates, services, SDK and engineering tests |
| `clavenar/specifications` | Public | Day one | Canonical architecture, security model, ABI and protocol specifications, ADRs and document releases |
| `clavenar/website` | Public | Day one | Main website, public project information and documentation navigation |
| `clavenar/registry` | Public | When packages stabilize | Agent, skill, workflow and tool package catalog; manifests, examples and distribution metadata |
| `clavenar/examples` | Public | When examples need separate maintenance | Tutorials, SDK integrations and deployment examples; initially keep them in Core |
| `clavenar/sdk-python` | Public | Independent SDK release | Python client SDK and language-specific packaging |
| `clavenar/sdk-typescript` | Public | Independent SDK release | TypeScript client SDK and language-specific packaging |
| `clavenar/research` | Public | As experiments justify it | Research papers, isolated prototypes and published benchmark studies |
| `clavenar/enterprise` | Private | When commercial extensions exist | Enterprise management, integrations, fleet governance and commercial reporting |
| `clavenar/cloud` | Private | When managed infrastructure exists | Hosted control plane, tenant management, managed Goal Machine orchestration and billing |
| `clavenar/infrastructure` | Private | When operational deployments exist | Infrastructure-as-code, release and production operations; secret references, never plaintext credentials |

Create only the first three repositories initially. All other names are reserved design choices; repository availability and permissions are not verified by this document. Keep reference agents and the first language bindings in Core until moving them has a concrete benefit.

## 2. Main repository: `clavenar/core`

The primary repository contains the implementation and executable test suites. The following is the **target layout**, not the day-one directory structure.

```text
core/
├── README.md
├── LICENSE
├── NOTICE
├── CONTRIBUTING.md
├── SECURITY.md
├── GOVERNANCE.md
├── ROADMAP.md
├── CHANGELOG.md
├── Cargo.toml
├── Cargo.lock
├── rust-toolchain.toml
├── rustfmt.toml
├── deny.toml
├── justfile
├── .cargo/
│   └── config.toml
├── .github/
│   ├── workflows/
│   ├── ISSUE_TEMPLATE/
│   └── CODEOWNERS
│
├── native/                 # Native Rust OS and architecture support
├── hosted/                 # Linux execution backend
├── virtual/                # Host-side VM lifecycle and guest integration
├── crates/                 # Shared architecture-independent Rust libraries
├── services/               # Shared user-space service implementations
├── userspace/              # Native init, libraries, shell and tools
├── agents/                 # Reference agents and executable scenarios
├── sdk/                    # Initial language bindings and SDK examples
├── cli/                    # clv management command
├── protocols/              # Machine-readable ABI and protocol schemas
├── platforms/              # Machine and board definitions
├── images/                 # Image manifests and construction recipes
├── tests/                  # Cross-component and cross-backend test harnesses
├── benchmarks/             # Reproducible comparative workloads
├── fuzz/                   # Security-sensitive parser and boundary fuzzing
├── verification/           # Formal models, proofs and invariant checks
├── examples/               # End-to-end demonstrations and tutorials
├── docs/                   # Implementation notes and build instructions
├── tools/                  # Developer, release and documentation tooling
└── xtask/                  # Rust build/test/image orchestration
```

`native/` owns the kernel, `userspace/` owns native applications, and `services/` owns reusable user-space service implementations. Do not keep separate copies of `modeld`, `mcpd` or other shared services under each deployment tree. Platform-specific integration belongs in adapters. The CLI presents a single logical API over the supported backends.

## 3. Native operating system

```text
native/
├── kernel/
│   ├── Cargo.toml
│   └── src/
│       ├── main.rs
│       ├── boot.rs
│       ├── panic.rs
│       ├── arch/
│       │   ├── mod.rs
│       │   ├── riscv64/
│       │   │   ├── boot.S
│       │   │   ├── sbi.rs
│       │   │   ├── trap.rs
│       │   │   ├── context.rs
│       │   │   ├── sv39.rs
│       │   │   └── timer.rs
│       │   ├── x86_64/
│       │   │   ├── boot.rs
│       │   │   ├── gdt.rs
│       │   │   ├── idt.rs
│       │   │   ├── apic.rs
│       │   │   ├── paging.rs
│       │   │   ├── context.rs
│       │   │   └── syscall.rs
│       │   └── aarch64/      # Later port
│       ├── memory/
│       │   ├── frame.rs
│       │   ├── allocator.rs
│       │   ├── address_space.rs
│       │   ├── mapping.rs
│       │   └── usercopy.rs
│       ├── process/
│       ├── scheduler/
│       ├── interrupt/
│       ├── time/
│       ├── syscall/
│       ├── elf/
│       ├── object/
│       ├── capability/
│       ├── goal/
│       ├── agent/
│       ├── budget/
│       ├── contract/
│       ├── security/
│       ├── ipc/
│       ├── event/
│       ├── reflex/
│       ├── audit/
│       ├── fs/
│       │   ├── vfs.rs
│       │   ├── ramfs.rs
│       │   ├── inode.rs
│       │   ├── directory.rs
│       │   ├── transaction.rs
│       │   └── snapshot.rs
│       └── drivers/
│           ├── uart/
│           ├── virtio/
│           ├── block/
│           ├── network/
│           └── entropy/
├── hal/                      # CPU, paging, interrupt and timer interfaces
├── bootloader/               # Own boot adapters, when needed
└── linker/
    ├── riscv64.ld
    ├── x86_64.ld
    └── aarch64.ld
```

### 3.1 Kernel responsibilities

| Module | Responsibility |
|---|---|
| `object` | Protected object identities, references, generation numbers and lifetimes |
| `capability` | Rights checks, delegation, attenuation and revocation |
| `goal`, `agent` | Protected identities, ownership, cancellation and lifecycle primitives |
| `budget` | Resource reservations, consumption accounting and bounded delegation |
| `contract` | Enforcement of supported deterministic, machine-readable restrictions |
| `security` | Security labels and supported information-flow mechanisms |
| `reflex` | Verified bounded policy execution, restricted helpers and event dispatch |
| `fs` | Resource mediation and supported transactional filesystem mechanisms |
| `audit` | Authoritative kernel events; durable storage is a user-space/storage responsibility |

The kernel remains deterministic in enforcement. Natural-language interpretation, LLM inference, agent prompt assembly, skill retrieval, JSON-RPC/MCP, HTTP and TLS belong in user space. A user-space service is still part of the trusted computing base when its compromise can violate a guarantee.

Architecture-specific code implements boot, page tables, trap entry, interrupts, timers, context switching and syscall entry/return. Scheduler policy, object management, capability semantics and goal logic should be architecture-neutral. Device transport code (e.g. VirtIO MMIO and PCI) should be separated from device-specific logic. The AMD64 port starts during native v0.3; its completion must not block the RV64 release.

## 4. Shared Rust crates

Crates represent proposed long-term boundaries, not a requirement to create dozens of packages immediately. Start with modules in a few crates; extract them when reuse, review boundaries or independent tests justify it.

```text
crates/
├── clv-types/               # IDs, units, errors and common value types
├── clv-abi/                 # Stable syscall numbers and binary layouts
├── clv-object/              # Shared object model and reference semantics
├── clv-capability/          # Rights and attenuation semantics
├── clv-goal/                # Goal model and lifecycle rules
├── clv-agent/               # Agent identity and metadata types
├── clv-budget/              # Reservation and hierarchical accounting
├── clv-contract/            # Goal Contract schema and deterministic checks
├── clv-policy/              # Authorization rules and policy evaluation
├── clv-effect/              # Effects, classification and effect budgets
├── clv-label/               # Confidentiality and integrity labels
├── clv-event/               # Event types and subscription contracts
├── clv-reflex/              # Shared reflex semantics and compiler-facing API
├── clv-reflex-bytecode/     # Versioned restricted instruction format
├── clv-reflex-verifier/     # Structural safety and bounded execution checks
├── clv-transaction/         # Transaction semantics and publication contracts
├── clv-audit/               # Event schemas and provenance records
├── clv-protocol/            # Versioned IPC and host–guest wire formats
├── clv-execution/           # Common backend interfaces and feature discovery
├── clv-runtime/             # Agent runtime and lifecycle coordination
├── clv-workflow/            # Structured workflow execution
├── clv-inference/           # Provider-independent inference interfaces
├── clv-mcp/                 # MCP client/server integration interfaces
├── clv-tool/                # Tool manifests, registry and invocation model
├── clv-package/             # Agent manifests, skills and package loading
└── clv-sdk/                 # Public Rust SDK
```

### 4.1 Dependency rules

```text
CLI / SDK / applications
          |
          v
Runtime / workflows / tool and inference integrations
          |
          v
Execution interfaces and platform-specific backends
          |
          v
Deterministic core: goals, rights, contracts, budgets, events, reflexes
          |
          v
Primitive types and ABI

Native kernel ---------> deterministic core + ABI + HAL
```

The arrow means "may depend on." The deterministic core must not depend on the kernel, Linux, networking, Tokio, provider SDKs or application services. Prefer `no_std` for the core, using `alloc` only where supported. Kernel-usable code must have bounded resource behavior and explicit failure paths; it need not be completely allocation-free unless a particular call context requires it. Keep OS-dependent policy enforcement in the relevant backend or broker. Share schema, types, pure algorithms and test vectors, not trusted handles or enforcement state between deployments.

Use stable-width integer representations and explicitly specified layouts for ABI and wire data. Do not expose Rust's default ABI, references or raw kernel pointers as public interfaces. Use separate wire representations rather than assuming in-memory C layouts are appropriate for all protocols.

### 4.2 Goal Contract Enforcement Architecture (GCEA)

```text
crates/clv-contract/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── contract.rs
    ├── scope.rs
    ├── constraint.rs
    ├── invariant.rs
    ├── precondition.rs
    ├── postcondition.rs
    ├── effect.rs
    ├── limits.rs
    ├── validation.rs
    └── digest.rs

services/contractd/
├── Cargo.toml
├── src/
│   ├── compiler.rs
│   ├── validator.rs
│   ├── authorization.rs
│   ├── plan.rs
│   ├── executor.rs
│   ├── postcondition.rs
│   └── publication.rs
└── tests/
```

Natural-language goals can generate proposed contracts in user space, but cannot grant authority. A trusted authorizer approves the contract against existing rights. Resource handlers enforce operation-specific effects, limits and live capabilities. Independent checks evaluate supported preconditions, invariants and postconditions; publication must bind the exact validated change set, base version and approval. Use canonical serialization for operation digests, replay protection and version-aware approval checks. The planner is untrusted. The kernel need not understand semantic correctness, and a model-generated plan is not proof of safety.

## 5. Clavenar Hosted

```text
hosted/
├── clavenard/
│   └── src/
│       ├── main.rs
│       ├── config.rs
│       ├── api.rs
│       ├── daemon.rs
│       ├── supervisor.rs
│       └── state.rs
├── linux-backend/
│   └── src/
│       ├── process.rs
│       ├── cgroup.rs
│       ├── namespace.rs
│       ├── seccomp.rs
│       ├── landlock.rs
│       ├── pidfd.rs
│       ├── network.rs
│       └── sandbox.rs
├── sandbox-launcher/
├── storage/
│   ├── overlay/
│   ├── workspace/
│   └── snapshot/
├── persistence/             # SQLite initially; optional other backends
├── wasm-runner/
├── reflex-runner/
├── api-server/
└── deployment/
    ├── systemd/
    ├── packages/
    └── containers/
```

`clavenard` is the control plane. The Linux backend maps logical operations onto namespaces, cgroups v2, seccomp, filesystem restrictions and authenticated broker communication. The sandbox launcher is narrowly privileged and separated from untrusted workloads. The Hosted implementation must prevent direct syscall, network and credential bypass of the SDK's policy checks. It must disclose which guarantees it can actually provide, including limitations of revoking already-open descriptors and publishing filesystem overlays atomically.

Start as a modular monolith with distinct logical service boundaries and minimal necessary isolated helper processes. Do not deploy every prospective daemon separately before there is a security, reliability or scaling reason. No custom Linux kernel module is required by this plan.

## 6. Clavenar Virtual and Goal Machines

```text
virtual/
├── clavenar-vmd/            # Host-side VM management service
├── goal-machine/
│   └── src/
│       ├── specification.rs
│       ├── allocation.rs
│       ├── lifecycle.rs
│       ├── provisioning.rs
│       ├── cancellation.rs
│       └── teardown.rs
├── backends/
│   ├── qemu/                # Initial QEMU/TCG and QEMU/KVM backend
│   ├── firecracker/         # Candidate after guest compatibility tests
│   └── native-kvm/          # Adapter to optional Clavenar VMM
├── vmm/                     # Optional direct-KVM Rust VMM, not MVP dependency
│   └── src/
│       ├── kvm.rs
│       ├── memory.rs
│       ├── vcpu.rs
│       ├── boot.rs
│       ├── interrupts.rs
│       ├── exit.rs
│       ├── lifecycle.rs
│       └── devices/
│           ├── console.rs
│           ├── block.rs
│           ├── rng.rs
│           └── transport.rs
├── guest-bridge/            # Native user-space client / guest adapter
├── host-bridge/             # Host-side bounded service transport
├── image-manager/
├── artifact-manager/
├── checkpoint-manager/
└── machine-pool/
```

The Goal Machine manager creates goal-scoped VMs, provisions approved inputs, allocates host resources, manages leases and cancellation, and exports authorized artifacts. A `VmBackend` interface encapsulates VMM selection and exposes actual supported features. QEMU TCG is the RV64 development environment; QEMU/KVM on AMD64 is the first accelerated commercial deployment target. Firecracker and a custom VMM using `/dev/kvm` directly are optional later work, gated by compatibility, benchmarks and security review.

Guest agents execute on the **same Clavenar native kernel** as Bare Metal. Do not fork the guest kernel into another product. The bridge should use a bounded, authenticated service protocol over suitable virtual devices such as VirtIO-console or vsock. Keep the no-NIC deployment option; remote inference and MCP can use the shared host brokers without guest TCP/IP. Host authorization must derive from the VM session and independently verified policy, not an untrusted guest-supplied goal ID. Run VMM workers separately with restricted host privileges and resource limits. No arbitrary host command execution or generic network proxy is required for the initial bridge.

## 7. Shared system services

```text
services/
├── agentd/                  # Agent lifecycle and execution coordination
├── goald/                   # Goal orchestration
├── modeld/                  # Inference routing, reservations and accounting
├── mcpd/                    # MCP client/server and restricted tool gateway
├── toold/                   # Tool discovery and invocation
├── policyd/                 # Authorization and information-flow policies
├── contractd/               # Goal contracts and independent verification
├── approvald/               # Human authorization, exact-operation binding
├── credentiald/             # Protected credential-backed operations
├── budgetd/                 # External token, cost and action accounting
├── workflowd/               # Deterministic workflow orchestration
├── reflexd/                 # Reflex compilation, testing and management
├── eventd/                  # Higher-level event routing
├── auditd/                  # Audit collection and durable records
├── snapshotd/               # Transaction/snapshot management
├── checkpointd/             # Logical agent checkpointing and recovery
├── registryd/               # Agent and tool registry
├── netd/                    # Native networking service
└── identityd/               # User and service identity
```

These are logical boundaries; deployment may consolidate implementations while isolating credentials and privileged operations. Hosted services may use mature Rust libraries and Linux networking. Native services use native syscalls/IPC and compatible libraries; do not assume Linux or Tokio-dependent binaries already run on Native. Virtual may run a guest front end and a trusted host back end of the same logical service. The shared `modeld` provider adapters and MCP gateways should not be rewritten for each environment.

## 8. Native userland, CLI and SDK

```text
userspace/
├── init/
├── libc/                    # Small compatibility library, if needed
├── libclv/                  # Native syscall wrappers and runtime support
├── shell/
├── coreutils/
│   ├── ls/
│   ├── cat/
│   ├── cp/
│   ├── mv/
│   ├── rm/
│   ├── mkdir/
│   ├── grep/
│   ├── find/
│   ├── sort/
│   ├── uniq/
│   ├── wc/
│   └── sha256sum/
├── sysutils/
│   ├── ps/
│   ├── kill/
│   ├── free/
│   ├── uptime/
│   └── mount/
├── netutils/
│   ├── ping/
│   ├── curl/
│   └── dig/
└── agentutils/              # Native goal, agent, capability, contract,
                             # reflex, audit, snapshot and MCP utilities

cli/                         # clv executable, with host/native adapters
sdk/
├── python/                  # Initial Python binding/client
└── typescript/              # Initial TypeScript client
```

The public Rust SDK lives in `crates/clv-sdk/`; `sdk/` contains other language bindings until independently maintained. Ordinary native ELF programs are first-class workloads. Tools can publish structured manifests for discovery by agents and selective exposure through MCP. Portable WebAssembly/WASI support is later work, not a dependency of the first native release. Do not equate supporting ELF with full Linux binary compatibility or full POSIX support.

## 9. Reference agents and packages

```text
agents/
├── supervisor/
├── diagnostics/
├── file-organizer/
├── code-reviewer/
├── resource-monitor/
├── security-guardian/
└── recovery/
```

Each reference agent follows a common definition layout. For example:

```text
agents/file-organizer/
├── README.md
├── agent.toml
├── INSTRUCTIONS.md
├── skills/
├── workflows/
├── contracts/
│   └── organize-documents.toml
├── policies/
│   ├── capabilities.toml
│   ├── network.toml
│   └── approvals.toml
├── tools/
├── reflexes/
└── tests/
    ├── normal-operation.yaml
    ├── malicious-document.yaml
    ├── capability-violation.yaml
    └── invalid-result.yaml
```

Instructions, skills and workflows define behavior; policies and contracts request authority, not grant it. Pin approved definition versions, authenticate package provenance and keep runtime state separate. Reference agents in Core are maintained with their conformance scenarios. The later public `registry` distributes independently versioned packages or points to approved upstream releases; avoid competing copies of the same canonical package.

## 10. ABI, protocols and platform targets

```text
protocols/
├── syscall-abi/
│   └── v1/
│       ├── syscalls.toml
│       ├── errors.toml
│       └── types.toml
├── host-guest/
│   └── v1/
├── agent-definition/
├── goal-contract/
├── capability/
├── inference/
├── tool-manifest/
├── reflex-bytecode/
├── checkpoint/
├── audit/
└── management-api/

platforms/
├── qemu-riscv64-virt/
├── qemu-x86_64-q35/
├── qemu-x86_64-microvm/
├── kvm-x86_64/
├── firecracker-x86_64/
├── qemu-aarch64-virt/
├── firecracker-aarch64/
└── hardware/
    ├── riscv64/
    └── aarch64/

images/
├── manifests/
├── native/
└── goal-machine/
```

Core owns the machine-readable schemas and generated code consumed by its builds and tests. `specifications` owns normative semantic documentation and protocol explanations; its release identifies the exact schema revision from Core. Pin compatible revisions in both repositories and check them in CI rather than maintaining duplicate editable schemas. Where generation is used, identify the canonical schema and make changes there first. Initial experimental versions need not promise ABI stability; establish explicit compatibility rules before stable releases.

Platform definitions are separate from CPU architecture implementations. They describe firmware, memory map, device discovery, transport, boot ABI, linker settings and image construction for a particular machine. RV64/OpenSBI under QEMU `virt` is the initial reference platform. Native AMD64 support is needed for efficient KVM guest execution on compatible Intel/AMD hosts. Support Firecracker and other VMMs only after their actual boot/device contracts have been implemented and tested.

## 11. Tests, fuzzing, verification and benchmarks

```text
tests/
├── unit/
├── integration/
├── kernel/
├── boot/
├── syscall/
├── filesystem/
├── capabilities/
├── contracts/
├── budgets/
├── transactions/
├── ipc/
├── reflexes/
├── inference/
├── mcp/
├── hosted/
├── virtual/
├── conformance/
└── adversarial/
    ├── prompt-injection/
    ├── capability-escalation/
    ├── information-leakage/
    ├── malicious-agent/
    ├── malicious-mcp-server/
    ├── budget-bypass/
    ├── approval-replay/
    ├── transaction-race/
    ├── malicious-reflex/
    └── hostile-guest/

fuzz/
├── elf-loader/
├── syscall-decoder/
├── reflex-verifier/
├── protocol-parser/
├── mcp-parser/
├── filesystem/
└── virtio-descriptors/

verification/
├── capability-model/
├── goal-lifecycle/
├── budget-accounting/
├── transaction-model/
├── reflex-termination/
└── information-flow/

benchmarks/
├── kernel/
│   ├── syscall-latency/
│   ├── context-switch/
│   └── ipc-throughput/
├── agent/
├── security/
├── reflex/
├── transaction/
├── inference/
├── virtual/
│   ├── boot-time/
│   ├── vm-density/
│   └── memory-footprint/
├── baselines/
│   ├── linux-native/
│   ├── linux-hardened/
│   └── linux-microvm/
└── reports/
```

Unit tests normally live alongside crate/module implementations; root-level tests hold integration fixtures, shared test harnesses and external conformance scenarios. A common conformance suite verifies authority attenuation, goal cancellation, hierarchical budget accounting, protected instruction versions, operation-bound approvals, transaction conflict checks and no unauthorized publication. Include deliberately malicious native programs and hostile VM input, not only adversarial prompts. Verify contract postconditions independently of the model. Backends must reject mandatory guarantees they cannot enforce; report support and failure honestly.

For benchmark comparisons use equivalent workloads, models, security policies, architectures and resource allocations; compare minimal hardened Linux guests with Clavenar guests on matching KVM hosts and VMMs. Include host broker overhead. Cross-ISA TCG is suitable for functional tests, not fair native performance comparisons. Formal models and fuzzing are assurance tools, not a claim that the entire OS has been formally verified.

## 12. Public specifications repository

```text
specifications/
├── README.md
├── LICENSE
├── CHANGELOG.md
├── architecture/
│   ├── architecture-and-vision.md
│   ├── kernel-design.md
│   ├── hosted-design.md
│   ├── virtual-design.md
│   └── goal-machines.md
├── security/
│   ├── threat-model.md
│   ├── capability-model.md
│   ├── goal-contracts.md
│   ├── information-flow.md
│   └── security-invariants.md
├── standards/
│   ├── syscall-abi.md
│   ├── agent-definition.md
│   ├── goal-contract.md
│   ├── reflex-bytecode.md
│   └── host-guest-protocol.md
├── engineering/
│   └── repository-layout.md
├── adr/
│   ├── 0001-rust.md
│   ├── 0002-riscv-first.md
│   ├── 0003-capability-security.md
│   ├── 0004-goals-as-kernel-objects.md
│   ├── 0005-reflex-architecture.md
│   ├── 0006-mcp-in-userspace.md
│   ├── 0007-amd64-early-port.md
│   ├── 0008-goal-machines.md
│   └── 0009-goal-contracts.md
├── assets/
│   └── diagrams/
├── templates/               # Preserve the established v0.5 visual style
├── tools/                   # Reproducible Markdown-to-PDF build
└── releases/                # Release manifests; PDF assets or links to tags
```

An ADR (Architecture Decision Record) captures context, decision, alternatives, consequences and status. ADRs are append-only in spirit; supersede prior decisions explicitly when the architecture changes.

Keep the architecture document's canonical Markdown source and diagram assets together. Preserve the established document styling, including typography, teal palette, cover, table of contents and running headers. Store a reproducible rendering template and its build configuration when available, rather than relying on manual PDF patches indefinitely. Publish immutable, versioned documents (e.g. `clavenar-architecture-and-vision-0.9.pdf`) through releases; include any required diagram assets with source archives. Document revisions are separate from software and protocol releases. This repository-layout document has its own revision series and may be stored as `engineering/repository-layout.md`.

## 13. Build, CI and release conventions

Use Cargo workspaces, `just` for human-facing commands and `xtask` for Rust build orchestration. Start with a root workspace and explicit members/default-members; select the kernel or user-space build target per invocation. If freestanding and host dependency/feature resolution conflicts arise, use nested independent workspaces rather than weakening the kernel's dependency boundary. Keep reproducible lockfiles for applications. Do not set a global bare-metal target for the entire repository. Keep architecture-specific linker flags and build profiles scoped appropriately.

Proposed developer commands (to be implemented):

```bash
git clone https://github.com/clavenar/core.git
cd core

just build riscv64
just run riscv64
just build x86_64
just run-kvm
just run-hosted
just run-virtual
just goal-machine
just test
just test-kernel
just test-security
just test-conformance
just fuzz
just benchmark
```

| CI workflow | Purpose |
|---|---|
| `rust.yml` | Formatting, Clippy, host unit tests and core dependency checks |
| `riscv64.yml` | Cross-compile RV64, boot QEMU and run kernel/userland tests |
| `x86_64.yml` | AMD64 cross-build, boot and guest conformance |
| `aarch64.yml` | Later ARM64 validation |
| `hosted.yml` | Linux integration and sandbox tests |
| `virtual.yml` | QEMU, host bridge, KVM and VM teardown tests |
| `security.yml` | Adversarial and cross-backend policy conformance |
| `fuzz.yml` | Parser, ABI and verifier fuzzing |
| `dependencies.yml` | Dependency licenses and vulnerability checks |
| `benchmarks.yml` | Reproducible performance regression runs |
| `release.yml` | Build artifacts, checksums, signatures and release metadata |
| `docs.yml` (Specifications) | Link checks, versioned PDF generation and artifact checks |

KVM-specific CI requires a compatible Linux runner with access to `/dev/kvm` and hardware/nested virtualization. Do not assume shared CI runners support this. Keep PR workloads isolated from release signing and production credentials.

Version Native, Hosted and Virtual independently, with explicit compatibility matrices for SDK, ABI, host–guest protocol, agent package and checkpoint formats. Document and release version numbers are not interchangeable. Illustrative artifacts:

```text
clavenar-riscv64-0.5.0.elf
clavenar-x86_64-0.5.0.img
clavenar-hosted-0.4.0-linux-x86_64.tar.gz
clavenar-virtual-0.5.0-linux-x86_64.tar.gz
clavenar-architecture-and-vision-0.9.pdf
clavenar-repository-layout-0.1.md
```

These names describe future artifact conventions, not delivered software. Publish a tagged source revision, build/run instructions, supported-platform/ABI declarations, acceptance-test results, known limitations, checksums and, when supported, signatures and dependency/SBOM information. Use Apache 2.0 as the proposed Core license, subject to dependency and governance review. Keep essential enforcement, core audit primitives and conformance tests in the public foundation; do not rely on commercial extensions to make the open-source kernel secure. Keep commercial infrastructure and operational secrets outside public repos.

## 14. Day-one bootstrap

Create the three initial repositories and populate only the directories needed for the first bootable implementation. This is a compact starting point, not a mandate to commit empty future crates or placeholder daemons.

```text
core/
├── README.md
├── LICENSE
├── SECURITY.md
├── ROADMAP.md
├── Cargo.toml
├── rust-toolchain.toml
├── justfile
├── .cargo/
├── .github/workflows/
├── native/
│   ├── kernel/
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── main.rs
│   │       ├── arch/riscv64/
│   │       ├── memory/
│   │       ├── syscall/
│   │       ├── drivers/
│   │       └── fs/
│   └── linker/riscv64.ld
├── crates/
│   ├── clv-types/
│   ├── clv-abi/
│   └── clv-capability/      # When the initial rights model is introduced
├── userspace/              # Init, syscall wrappers and shell as they emerge
├── platforms/qemu-riscv64-virt/
├── tests/
├── docs/
└── xtask/
```

| Native milestone | Grow these areas | Parallel Hosted / Virtual work |
|---|---|---|
| Internal prototype / v0.1 Foundation | RV64 boot, allocator, Sv39, safe copying, traps, ELF, scheduler, shell and RamFS | Reproducible QEMU launch, shared types and basic host harness |
| v0.2 Secure Core | Object tables, capabilities, confined filesystem, IPC, events | Linux sandbox controls and common security tests |
| v0.3 Agent Core | Goals, agents, hierarchical budgets, initial contract restrictions, definitions, mock inference and audit; begin AMD64 port | Hosted runtime and broker development; guest bridge preparation |
| v0.4 Agentic MVP | Transactions, exact approvals, local MCP and real inference via host bridge | Reuse inference/MCP services; introduce restricted Goal Contract validation and independent checks for supported effects |
| v0.5 Reflex | Bounded reflex prototype and replay; target AMD64 guest conformance | QEMU/KVM Goal Machine MVP, cancellation, mediated egress and artifact export |
| v0.6 Persistence | Durable storage, recovery and checkpoints | Durable artifacts, accounting and pending external-operation reconciliation |
| v0.7 Standalone | Native network service, IP/TCP, HTTPS, remote MCP and inference | Expanded VM conformance; retain no-NIC mode |
| v0.8+ Expansion | Stronger information-flow assurance, ARM64 and portable Wasm | Optional Firecracker/direct-KVM VMM, pools and distributed management as justified |
| v1.0 Production | Evidence-based security, stability, compatibility and operational readiness | Independent Hosted and Virtual production release gates |

The detailed architecture and release criteria in the master architecture document remain authoritative for existing milestones. The GCEA module boundaries and contract-specific deliverables in this layout reflect the subsequent design discussion; incorporate them into the master roadmap before treating them as committed release gates. Do not require every optional backend to ship in the same release.

## 15. Engineering rules

1. Keep the native kernel independent of Linux and all LLM/network protocol libraries.
2. Share deterministic semantics and test vectors; enforce them separately for each backend.
3. Keep the goal as a protected resource with explicit lifetime, authority, budgets and cancellation.
4. Model instructions and manifests describe requested behavior; they do not confer privileges.
5. Use explicit effect restrictions, goal contracts, independent verification and staged publication where supported.
6. No backend silently weakens mandatory security requirements. Record unsupported capabilities and fail closed.
7. Virtual always executes the actual native kernel; Hosted does not substitute for native milestones.
8. Build the minimal RV64 foundation first. Bring AMD64 forward for hardware-accelerated Goal Machines; ARM64 and a custom VMM are later, conditional work.
9. Maintain one authoritative source for each schema, specification and implementation; keep versions pinned across repositories.
10. Prefer small, reproducible deliverables and security tests to premature crate proliferation or a large set of empty repositories.

---

**Recommended initial organization:** `clavenar/core`, `clavenar/specifications`, `clavenar/website`.

**Core principle:** Clavenar is one coherent autonomous execution platform with three deployment modes, not three unrelated projects.
