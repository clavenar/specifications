# Changelog

## Unreleased

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
