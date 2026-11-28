# Clavenar Specifications

This repository publishes the Clavenar architecture and engineering layout.

- [Architecture and vision, draft 0.31](architecture/architecture-and-vision.md) defines the goals, security model and release criteria.
- [Verified RV64 baseline](architecture/architecture-and-vision.md#21-rv64-prototype-and-verification-baseline) and [MVP scenario status](architecture/architecture-and-vision.md#241-demonstration-scenario) give reproducible QEMU examples for kernel 0.1.16, including reviewed publication, recovery, native MCP over a Linux route, shell-only UART input, silent guest-only RamFS entry and explicit display, confined tool streams and explicit collection of protected replies and completion details.
- [Repository layout, revision 0.21](engineering/repository-layout.md) locates the current native implementation and maps the planned components.
- [Changelog](CHANGELOG.md) records document revisions.

The `security/`, `standards/` and `adr/` directories are reserved for
standalone security specifications, protocol standards and decision records.
Architecture flow diagrams use Mermaid. The virtual filesystem namespace uses a text tree.
The document publishing toolchain and license are pending. [Archived source drafts](archive/imported-drafts/)
