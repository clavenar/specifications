# Clavenar Specifications

This repository publishes the Clavenar architecture and engineering layout.

- [Architecture and vision, draft 0.12](architecture/architecture-and-vision.md) defines the goals, security model and release criteria.
- [Verified RV64 baseline](architecture/architecture-and-vision.md#21-rv64-prototype-and-verification-baseline) and [MVP scenario status](architecture/architecture-and-vision.md#241-demonstration-scenario) document what runs in QEMU and how to reproduce it.
- [Repository layout, revision 0.3](engineering/repository-layout.md) locates the current native implementation and maps the planned components.
- [Changelog](CHANGELOG.md) records document revisions.

The `security/`, `standards/` and `adr/` directories are reserved for
standalone security specifications, protocol standards and decision records.
The architecture document embeds its four diagrams as Mermaid blocks. The
document publishing toolchain and license are pending. [Archived source drafts](archive/imported-drafts/)
