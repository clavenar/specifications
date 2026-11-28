# Clavenar — Repository Layout

**Document revision:** 0.6\
**Date:** 28 November 2026\
**Organization:** `clavenar`  
**Primary repository:** `clavenar/core`  
**Website:** `clavenar.io`  
**Related document:** [Clavenar — Architecture and Vision](../architecture/architecture-and-vision.md)

Section 2.1 records the sources for RV64 kernel 0.1.1 and `clv-abi` 0.0.19 at Core revision `42da788`. The target trees describe planned components. [Current Core tree](#21-current-core-tree) · [Build and boot](#131-build-and-boot-the-rv64-prototype)

> One execution model, multiple deployment environments, a shared security contract.

## 1. Repository strategy

Clavenar has three deployment modes. Native is the Rust kernel and userland shared by Bare Metal and Virtual.

- **Bare Metal** runs the native kernel directly on supported physical hardware, a later platform target.
- **Hosted** is the proposed Linux runtime for the common goal, agent and policy contracts.
- **Virtual** runs the native kernel as a VM guest. The RV64 prototype boots in QEMU today. The host-side Goal Machine manager and AMD64/KVM deployment are planned.

Use a public engineering monorepo, `clavenar/core`, for the shared implementation. Keep the public specifications and website separate; split other repositories only when independent ownership, release cadence, distribution or access control warrants it.

Native OS development is the primary engineering objective. The QEMU host broker handles model inference and brokered MCP calls. The guest supports MCP over local stdio and can discover and invoke the fixed `text.search` tool over native HTTPS on the private QEMU link. The proposed Hosted runtime would execute agents on Linux. The planned AMD64 Virtual target would execute the native kernel on Linux/KVM.

### 1.1 GitHub organization

| Repository | Intended visibility | Stage | Responsibility |
|---|---|---|---|
| `clavenar/core` | Public | In use | Native kernel and userland today. Planned home for Hosted, Virtual, shared services, SDK and engineering tests. |
| `clavenar/specifications` | Public | In use | Canonical architecture and engineering layout, with space for normative security and protocol documents and ADRs. |
| `clavenar/website` | Public | Scaffold | Project site content, assets and documentation navigation. |
| `clavenar/registry` | Public | When packages stabilize | Agent, skill, workflow and tool package catalog; manifests, examples and distribution metadata |
| `clavenar/examples` | Public | When examples need separate maintenance | Tutorials, SDK integrations and deployment examples; initially keep them in Core |
| `clavenar/sdk-python` | Public | Independent SDK release | Python client SDK and language-specific packaging |
| `clavenar/sdk-typescript` | Public | Independent SDK release | TypeScript client SDK and language-specific packaging |
| `clavenar/research` | Public | As experiments justify it | Research papers, isolated prototypes and published benchmark studies |
| `clavenar/enterprise` | Private | When commercial extensions exist | Enterprise management, integrations, fleet governance and commercial reporting |
| `clavenar/cloud` | Private | When managed infrastructure exists | Hosted control plane, tenant management, managed Goal Machine orchestration and billing |
| `clavenar/infrastructure` | Private | When operational deployments exist | Infrastructure-as-code, release and production operations; secret references, never plaintext credentials |

The first three repositories form the bootstrap. The remaining names are proposals. Keep reference agents and the first language bindings in Core until independent ownership or distribution justifies a split.

## 2. Main repository: `clavenar/core`

### 2.1 Current Core tree

The build embeds five user ELFs in seven fixed task slots. `echo` supplies the ordinary, supervisor and monitor slots. `netd` starts when a supported NIC initializes. The tree shows the source entry points for this revision. [Core at `42da788`](https://github.com/clavenar/core/tree/42da7882d096a969fb1c69c215fa6403e29e7bb6)

```text
core/
├── README.md                            # Prerequisites, kernel build and QEMU quickstart
├── ROADMAP.md                           # Release gates and links to working examples
├── rust-toolchain.toml                  # Rust 1.100.1 and the RV64 target
├── docs/
│   ├── qemu-walkthroughs.md             # Host and guest commands with expected results
│   └── native-syscall-abi.md            # RV64 registers, syscalls and wire formats
├── native/
│   ├── kernel/
│   │   ├── Cargo.toml
│   │   ├── Cargo.lock
│   │   ├── build.rs                     # Compile and embed the user ELFs
│   │   ├── build/definition.rs          # Validate and pin agent definitions
│   │   └── src/
│   │       ├── main.rs                  # Boot and scheduler setup
│   │       ├── programs.rs              # Embedded images and tool identities
│   │       ├── arch/riscv64/            # Entry, traps and timer
│   │       ├── drivers/                 # UART, RTC and VirtIO MMIO devices
│   │       │   └── virtio/              # Block, console, network and RNG drivers
│   │       ├── memory/                  # Frames, Sv39, ELF loading and user copies
│   │       ├── fs/ramfs.rs              # Paths, directories and file storage
│   │       ├── fs/ramfs/                # Handles, staged renames and durable records
│   │       ├── ipc.rs                   # Bounded typed messages
│   │       ├── ipc/pipe.rs              # Native streams
│   │       ├── host_bridge.rs           # Guest side of the restricted host channel
│   │       ├── scheduler.rs             # Task slots, launches, waits and events
│   │       ├── scheduler/goal.rs        # Goal and agent lifetimes
│   │       ├── scheduler/goal/          # Budgets, grants, monitoring, review and checkpoints
│   │       └── syscall/                 # Dispatch and network-service calls
│   └── linker/riscv64.ld
├── crates/clv-abi/src/                  # Shared syscall numbers, wire types and registries
├── platforms/qemu-riscv64-virt/
│   ├── host_bridge.py                   # QEMU launcher and SQLite request journal
│   ├── bridge_protocol.py               # Guest–host wire messages
│   ├── host_json.py                     # Strict JSON decoding for host responses
│   ├── ollama_backend.py                # Loopback model calls
│   ├── mcp_backend.py                   # Loopback MCP tool calls
│   ├── replay_backend.py                # Replay of recorded decisions and tool receipts
│   ├── network_fixture.py               # QEMU link, DNS and TLS walkthrough peers
│   └── mcp_fixture.py                   # Loopback MCP search server
└── userspace/
    ├── user.ld                          # User ELF layout
    ├── libclv/                          # RV64 syscall wrappers
    ├── shell/                           # clsh commands, help and audit display
    ├── hello/                           # Worker and local MCP client
    │   └── definitions/                 # find-greeting revisions 1–8
    ├── echo/                            # Ordinary child, supervisor and monitor
    ├── grep/                            # Native search tool and stdio MCP server
    └── netd/                            # smoltcp network service
        ├── http_probe.rs                # Plain HTTP client
        ├── http_response.rs             # Shared bounded HTTP response parsing
        ├── tls_probe.rs                 # DNS, TLS and goal-authorized HTTPS transport
        └── tls_probe/mcp.rs             # Native MCP initialization, discovery and invocation
```

The QEMU platform contains the host broker, its backends and the host-side networking fixtures. The top-level `tests/`, `xtask/` and `.github/workflows/` directories are placeholders. The walkthroughs give runnable fixtures and expected output. [QEMU walkthroughs](https://github.com/clavenar/core/blob/42da7882d096a969fb1c69c215fa6403e29e7bb6/docs/qemu-walkthroughs.md) · [RV64 baseline](../architecture/architecture-and-vision.md#21-rv64-prototype-and-verification-baseline)

### 2.2 Target Core tree

This tree assigns ownership as Core grows. Sections 3–11 give the planned paths for its subsystems.

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
│       │   └── x86_64/
│       │       ├── boot.rs
│       │       ├── gdt.rs
│       │       ├── idt.rs
│       │       ├── apic.rs
│       │       ├── paging.rs
│       │       ├── context.rs
│       │       └── syscall.rs
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
    └── x86_64.ld
```

### 3.1 Kernel responsibilities

| Module | Responsibility |
|---|---|
| `object` | Protected object identities, references, generation numbers and lifetimes |
| `capability` | Rights checks, delegation, attenuation and revocation |
| `goal`, `agent` | Protected identities, ownership, cancellation and lifecycle primitives |
| `budget` | Local resource reservations and bounded delegation. Trusted brokers reserve and settle external spending. |
| `contract` | Enforcement of supported deterministic, machine-readable restrictions |
| `security` | Security labels and supported information-flow mechanisms |
| `reflex` (conditional) | Selected verified helpers and event dispatch if a privileged backend meets the evidence gates. Native v0.5 starts with bounded policies in user space. |
| `fs` | Resource mediation and supported transactional filesystem mechanisms |
| `audit` | Authoritative kernel events; durable storage is a user-space/storage responsibility |

The kernel remains deterministic in enforcement. Natural-language interpretation, LLM inference, agent prompt assembly, skill retrieval, JSON-RPC/MCP, HTTP and TLS belong in user space. A user-space service is still part of the trusted computing base when its compromise can violate a guarantee.

Architecture-specific code implements boot, page tables, trap entry, interrupts, timers, context switching and syscall entry/return. Scheduler policy, object management, capability semantics and goal logic should be architecture-neutral. Device transport code (e.g. VirtIO MMIO and PCI) should be separated from device-specific logic. The AMD64 port starts during native v0.3; its completion must not block the RV64 release.

## 4. Shared Rust crates

Crates represent proposed long-term boundaries, not a requirement to create dozens of packages immediately. Start with modules in a few crates; extract them when reuse, review boundaries or independent tests justify it.

```text
crates/
├── clv-types/               # IDs, units, errors and common value types
├── clv-abi/                 # Syscall numbers and shared ABI values
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
Runtime, workflows and tool integrations     Native and Hosted adapters
                     |                                  |
                     +--------------+-------------------+
                                    v
                         clv-execution interfaces
                                    |
                                    v
                 Shared goals, rights, contracts and budgets
                                    |
                                    v
                             Primitive types

Native kernel -> bounded shared code, clv-abi and architecture support
```

Arrows show source dependencies. The shared runtime calls the execution interface, each backend implements it, and the deployment selects the backend. Keep the deterministic core independent of Linux, the native kernel, networking, Tokio and provider SDKs. Prefer `no_std` for kernel-usable code, with bounded resource use and explicit failure paths. Backends and brokers own their live handles and enforcement state. They can share schemas, pure algorithms and conformance vectors. [Shared implementation contracts](../architecture/architecture-and-vision.md#316-shared-implementation-contracts)

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

A planner proposes a contract in user space. A trusted authorizer binds its version and permitted effects to the caller's delegable rights. Resource handlers check live grants, limits and preconditions at dispatch. The contract service coordinates independent checks on the staged result. The storage service orders publication against cancellation and checks the frozen diff, expected base, required postconditions and applicable approval. An unsupported mandatory check blocks publication. Use a canonical encoding to identify the reviewed operation. [Goal contracts](../architecture/architecture-and-vision.md#52-intent-based-security) · [Transactional publication](../architecture/architecture-and-vision.md#12-transactional-autonomy-and-versioned-filesystem)

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

For repository jobs, `hosted/storage/` supplies a pinned input snapshot and private writable workspace. The launcher keeps the original repository and shared writable Git metadata outside the sandbox. The trusted publication service checks the reviewed revision against the expected base. [Hosted workspaces and publication guarantees](../architecture/architecture-and-vision.md#315-workspaces-persistent-state-and-hosted-reflexes)

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

The current native network service lives in `userspace/netd/`. The `services/` paths above show the intended shared-service layout. [Native network service](https://github.com/clavenar/core/tree/42da7882d096a969fb1c69c215fa6403e29e7bb6/userspace/netd)

These are logical boundaries; deployment may consolidate implementations while isolating credentials and privileged operations. Hosted services may use mature Rust libraries and Linux networking. Native services use native syscalls/IPC and compatible libraries; do not assume Linux or Tokio-dependent binaries already run on Native. Virtual may run a guest front end and a trusted host back end of the same logical service. The shared `modeld` provider adapters and MCP gateways should not be rewritten for each environment.

## 8. Native userland, CLI and SDK

The native `clsh>` prompt belongs to `userspace/shell/`. File and child commands use `userspace/libclv/`. The `goal.rs` module and its `goal/` subdirectory handle lifecycle commands, options and audit display. The `cli/` tree below is the proposed cross-backend `clv` management CLI.

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
└── hardware/
    └── riscv64/

images/
├── manifests/
├── native/
└── goal-machine/
```

The prototype defines 78 native syscall numbers in `crates/clv-abi/src/lib.rs`. Other modules in the same crate define shared goal, audit, host-bridge and network records. The kernel dispatcher and user-space wrappers use these definitions. The planned `protocols/` tree holds machine-readable schemas. Core owns the schemas and generated code. Specifications defines their semantics and pins the matching Core revision. Specify compatibility rules before declaring an interface stable. [Syscall definitions](https://github.com/clavenar/core/blob/42da7882d096a969fb1c69c215fa6403e29e7bb6/crates/clv-abi/src/lib.rs) · [Native syscall ABI](https://github.com/clavenar/core/blob/42da7882d096a969fb1c69c215fa6403e29e7bb6/docs/native-syscall-abi.md)

Platform definitions are separate from CPU architecture implementations. They describe firmware, memory map, device discovery, transport, boot ABI, linker settings and image construction for a particular machine. RV64/OpenSBI under QEMU `virt` is the initial reference platform. Native AMD64 support is needed for efficient KVM guest execution on compatible Intel/AMD hosts. Support Firecracker and other VMMs only after their actual boot/device contracts have been implemented and tested.

## 11. Tests, fuzzing, verification and benchmarks

```text
tests/
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

The tree below assigns locations for future normative documents and release artifacts. The specifications index lists the published architecture and layout. [Published documents](../README.md)

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
├── archive/
│   └── imported-drafts/     # Original source documents
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
├── templates/               # Planned publishing templates
├── tools/                   # Reproducible Markdown-to-PDF build
└── releases/                # Release manifests; PDF assets or links to tags
```

An ADR (Architecture Decision Record) captures context, decision, alternatives, consequences and status. ADRs are append-only in spirit; supersede prior decisions explicitly when the architecture changes.

Edit the maintained Markdown in `architecture/` and `engineering/`. The archive retains the historical draft versions with the current CPU target scope. Git history preserves the original imports. Architecture flow diagrams are embedded in Markdown, and the virtual filesystem namespace uses a text tree. Store separate figure assets in `assets/diagrams/`. [Imported drafts](../archive/imported-drafts/)

The PDF publishing toolchain is planned. Its template should carry the intended typography, teal palette, cover, contents and running headers. A document release should identify the source commit, rendering configuration and figure dependencies so the published artifact can be reproduced. Architecture and repository layout each use their own document revision.

## 13. Build, CI and release conventions

### 13.1 Build and boot the RV64 prototype

From the Core repository root at `42da788`, build kernel 0.1.1 with Rust 1.100.1 and the `riscv64gc-unknown-none-elf` target pinned in `rust-toolchain.toml`. The command below boots the default image on QEMU `virt` with OpenSBI and the NIC disabled. [Prerequisites and quickstart](https://github.com/clavenar/core/blob/42da7882d096a969fb1c69c215fa6403e29e7bb6/README.md#build-and-run-the-rv64-kernel)

```sh
cargo rustc --locked --manifest-path native/kernel/Cargo.toml \
  --target riscv64gc-unknown-none-elf --target-dir target --release \
  -- -C "link-arg=-T${PWD}/native/linker/riscv64.ld" -C link-arg=--no-relax

qemu-system-riscv64 -machine virt -smp 1 -m 128M -bios default -nographic -nic none \
  -kernel target/riscv64gc-unknown-none-elf/release/clavenar-kernel
```

At this revision the serial console prints `Clavenar kernel booted (v0.1.1)`, five loaded ELF entries and `Sv39 paging enabled`. Type `help` at the `clsh>` prompt for shell commands. Press `Ctrl+a`, then `x`, to quit QEMU.

### 13.2 Planned tooling and releases

The target build design adds a root Cargo workspace, `just` recipes and `xtask`. Give each component an explicit target and keep bare-metal linker flags scoped to the kernel build. If host and freestanding dependencies need separate resolution, they can use independent workspaces. The following recipes and CI workflows are proposed.

```bash
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

| Proposed CI workflow | Purpose |
|---|---|
| `rust.yml` | Formatting, Clippy, host unit tests and core dependency checks |
| `riscv64.yml` | Cross-compile RV64, boot QEMU and run kernel/userland tests |
| `x86_64.yml` | AMD64 cross-build, boot and guest conformance |
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
clavenar-architecture-and-vision-0.14.pdf
clavenar-repository-layout-0.4.md
```

These names describe future artifact conventions, not delivered software. Publish a tagged source revision, build/run instructions, supported-platform/ABI declarations, acceptance-test results, known limitations, checksums and, when supported, signatures and dependency/SBOM information. Use Apache 2.0 as the proposed Core license, subject to dependency and governance review. Keep essential enforcement, core audit primitives and conformance tests in the public foundation; do not rely on commercial extensions to make the open-source kernel secure. Keep commercial infrastructure and operational secrets outside public repos.

## 14. Day-one bootstrap

For a new checkout, build the default RV64 image and run a goal with a read-only file grant. The QEMU walkthroughs then cover reviewed publication, checkpoint recovery, host-brokered model and MCP calls, and a private native network link. [Build and boot](#131-build-and-boot-the-rv64-prototype) · [Current Core tree](#21-current-core-tree)

Architecture Section 23 defines the native, Hosted and Virtual release gates. Section 24 maps the six file-organizer scenarios to current evidence. [Native release milestones](../architecture/architecture-and-vision.md#232-native-os-release-milestones) · [Cross-product release gates](../architecture/architecture-and-vision.md#234-cross-product-release-gates) · [MVP evidence](../architecture/architecture-and-vision.md#241-demonstration-scenario)

## 15. Engineering rules

1. Keep the native kernel independent of Linux and all LLM/network protocol libraries.
2. Share deterministic semantics and test vectors; enforce them separately for each backend.
3. Keep the goal as a protected resource with explicit lifetime, authority, budgets and cancellation.
4. Model instructions and manifests describe requested behavior; they do not confer privileges.
5. Use explicit effect restrictions, goal contracts, independent verification and staged publication where supported.
6. No backend silently weakens mandatory security requirements. Record unsupported capabilities and fail closed.
7. Virtual always executes the actual native kernel; Hosted does not substitute for native milestones.
8. Build the minimal RV64 foundation first. Target AMD64 for hardware-accelerated Goal Machines. Evaluate a custom VMM after measuring the QEMU/KVM deployment.
9. Maintain one authoritative source for each schema, specification and implementation; keep versions pinned across repositories.
10. Prefer small, reproducible deliverables and security tests to premature crate proliferation or a large set of empty repositories.

---

**Core principle:** Clavenar is one coherent autonomous execution platform with three deployment modes, not three unrelated projects.
