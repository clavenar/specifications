# Clavenar Specifications

This repository publishes the Clavenar architecture and engineering layout.

- [Architecture and vision, draft 0.18](architecture/architecture-and-vision.md) defines the goals, security model and release criteria.
- [Verified RV64 baseline](architecture/architecture-and-vision.md#21-rv64-prototype-and-verification-baseline) and [MVP scenario status](architecture/architecture-and-vision.md#241-demonstration-scenario) give reproducible QEMU examples for kernel 0.1.3, including reviewed publication, recovery and goal-authorized MCP calls to two HTTPS peers through an IPv4 gateway.
- [Repository layout, revision 0.8](engineering/repository-layout.md) locates the current native implementation and maps the planned components.
- [Changelog](CHANGELOG.md) records document revisions.

The `security/`, `standards/` and `adr/` directories are reserved for
standalone security specifications, protocol standards and decision records.
Architecture flow diagrams use Mermaid. The virtual filesystem namespace uses a text tree.
The document publishing toolchain and license are pending. [Archived source drafts](archive/imported-drafts/)
