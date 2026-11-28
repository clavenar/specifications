# Changelog

## Unreleased

- Architecture draft 0.27 records explicit display of queued and saved
  guest-only replies on kernel 0.1.12 and `clv-abi` 0.0.25. [Core at `717b50f`](https://github.com/clavenar/core/tree/717b50f2daa670bc5c9b7c457df7e55ebad628cf)
- Repository layout 0.17 locates the receive checks and retained policy.
- Architecture draft 0.26 records writable guest-only RamFS inputs and
  publication denial on kernel 0.1.11 and `clv-abi` 0.0.24. [Core at `9184b2b`](https://github.com/clavenar/core/tree/9184b2bc671f9e0dda59b4c958a86ee80676ce6a)
- Repository layout 0.16 locates file policy, the shell-only open mode,
  and the grant and read checks.
- Architecture draft 0.25 records shell-only UART input and stdin pipe
  verification on kernel 0.1.10. [Core at `253502e`](https://github.com/clavenar/core/tree/253502ea899faad15c23ecc4e85f1f538cc1d470)
- Repository layout 0.15 locates the scheduler identity and syscall guard
  for raw UART input.
- Architecture draft 0.24 records guest-only rename restrictions and staging
  audit entries that omit the requested name on kernel 0.1.9 and `clv-abi`
  0.0.23. [Core at `abd4b08`](https://github.com/clavenar/core/tree/abd4b0851bff361d61b84d2ca3847003e1478b0d)
- Repository layout 0.14 locates the admission, staging and commit
  checks and the shared audit record.
- Architecture draft 0.23 records serial write denial, restricted exit
  and fault logs and trusted shell replies for guest-only goals on
  kernel 0.1.8. [Core at `53813fc`](https://github.com/clavenar/core/tree/53813fcdce6b0a4daacb88f6acbad1170cb677bf)
- Repository layout 0.13 locates the serial write, goal membership and
  trap diagnostic checks.
- Architecture draft 0.22 records the protected RamFS input, local MCP
  search and external dispatch denials on kernel 0.1.7 and `clv-abi` 0.0.22. [Core at `a6200b0`](https://github.com/clavenar/core/tree/a6200b01dda4ce72f624ba06327d8045dd4bb52b)
- Repository layout 0.12 locates the boot file metadata and the grant
  and read checks.
- Architecture draft 0.21 records guest-only goal egress and local
  checkpoint recovery on kernel 0.1.6 and `clv-abi` 0.0.22. [Core at `156abc2`](https://github.com/clavenar/core/tree/156abc2cd40d504a0b896dfa4973df6824b14c08)
- Repository layout 0.11 locates the native HTTPS and host-broker
  admission checks and the checkpoint recovery path.
- Architecture draft 0.20 records the Linux-routed MCP call and DNS, CA
  and transmit-budget denials on kernel 0.1.5. [Core at `cab3ff8`](https://github.com/clavenar/core/tree/cab3ff890f457dc91f0945c536c25dbae1055a35)
- Repository layout 0.10 links the TAP walkthrough and the reuse of the
  existing MCP fixture as the HTTPS service.
- Architecture draft 0.19 records the kernel-enforced HTTPS transmit
  budget, QEMU frame accounting and cleanup after exhaustion on kernel
  0.1.4 and `clv-abi` 0.0.21. [Core at `beef264`](https://github.com/clavenar/core/tree/beef26455e08273389673deec71e6ff2f3e50082)
- Repository layout 0.9 locates the goal and syscall code that enforces
  the native HTTPS transmit budget.
- Architecture draft 0.18 records per-goal HTTPS peer selection on kernel
  0.1.3 and `clv-abi` 0.0.20, including QEMU checks for independent DNS pins
  and trust anchors. [Core at `d2e6ce5`](https://github.com/clavenar/core/tree/d2e6ce5e9e042ca4d8482f75d5a91fee5a6b179e)
- Repository layout 0.8 locates the compiled HTTPS peer table and the
  walkthrough for selecting between its peers.
- Architecture draft 0.17 records the static IPv4 and HTTPS profile and
  the routed MCP walkthrough on kernel 0.1.2. [Core at `71559ef`](https://github.com/clavenar/core/tree/71559efde4b37721543d4a6fd1942eeaa1b26c48)
- Repository layout 0.7 locates the network profile builder and the default
  QEMU profile.
- Architecture draft 0.16 records the goal-authorized native MCP search on
  kernel 0.1.1 and `clv-abi` 0.0.19. It describes the fixed endpoint and remaining Phase 15 work. [Core at `42da788`](https://github.com/clavenar/core/tree/42da7882d096a969fb1c69c215fa6403e29e7bb6)
- Repository layout 0.6 locates the QEMU network fixtures and the native MCP client.
- Architecture draft 0.15 uses kernel 0.0.99 and `clv-abi` 0.0.17. It records goal-authorized HTTPS and native MCP discovery, and identifies remote tool invocation as remaining phase 15 work. [Core at `46eaeb5`](https://github.com/clavenar/core/tree/46eaeb591c0957c40e0140c1e3cafad35d6eeb62)
- Repository layout 0.5 records the native HTTP and TLS sources and uses the pinned Rust 1.100.1 toolchain in the RV64 build instructions.
- Architecture draft 0.14 records kernel 0.0.93 and `clv-abi` 0.0.14 at
  Core `9559b5b`. It updates the six MVP scenarios, checkpoint recovery,
  host-brokered Ollama and MCP, and the phase 15 networking work.
- Repository layout 0.4 maps the QEMU broker, `netd` and the five native ELF
  images. Both maintained documents and the archived target lists now specify
  RV64 as the reference architecture and AMD64 as the planned second port.
  Archived figure links point to the diagrams in the maintained architecture.
- Architecture draft 0.13 replaces text diagrams with Mermaid figures for goal
  supervision, resource layouts, development tracks, reflex and MCP execution,
  Hosted services, Goal Machines and inference routes.
- Architecture draft 0.12 records kernel 0.0.35 at `f0b1003` and the QEMU
  checks repeated on 0.0.36. It corrects the default RamFS authority, retains
  the historical 0.0.1 baseline and links evidence for the six native v0.4 scenarios.
- Repository layout 0.3 maps the four native user programs, goal and RamFS
  modules, QEMU walkthroughs and ABI reference at Core revision `cd78ad9`.
- Revised the repository layout to 0.2 with the checked-in RV64 source tree,
  working Cargo and QEMU commands, and milestones aligned with architecture draft 0.11.
- Bumped the architecture and vision document to draft 0.11.
- Revised architecture and vision to draft 0.10 with a reproducible RV64 baseline,
  inline diagrams, and aligned goal contract, publication and reflex release criteria.
- Added the architecture and vision draft 0.9 and repository layout revision 0.1.
