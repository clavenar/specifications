# Repository Guidelines

## Project Structure and Development

This workspace contains three independent Git repositories:

- `core/` holds the engineering scaffold. Kernel source belongs in
  `core/native/kernel/src/`, shared Rust code in `core/crates/`, and
  cross-component tests in `core/tests/`.
- `specifications/` holds architecture, engineering, security, protocol, and ADR
  documents. Diagram assets belong in `specifications/assets/diagrams/`.
- `website/` holds public site content and assets in `website/content/` and
  `website/assets/`.
- `docs/` holds the original drafts imported into `specifications/`. Edit the
  copies in `specifications/` for future document revisions.

Add setup instructions to each repository README when dependencies or
configuration are required locally.

Once build and test tooling is in place, list the exact commands contributors should run from the repository root. Include any prerequisites, the command to start the project locally, and commands for building, running tests, and checking formatting. Verify each command before documenting it.

## Coding Style and Naming

When editing existing code, follow the indentation, naming, and layout of nearby files. Once the project adopts a formatter or linter, use its checked-in configuration and document how to run it. Choose names that describe the purpose of each module, function, or file.

## Testing

When introducing a test framework, document where tests live, how test files are named, and how to run a single test and the full suite. Include any coverage threshold enforced by the project. For changes made before automated checks exist, record the manual verification steps in the pull request.

## Commits and Pull Requests

Start every commit subject with a lowercase letter and keep it short and descriptive. In each pull request, summarize the change and include the checks you ran. Link a relevant issue when one exists and attach screenshots for visible UI changes. Update this section when the project establishes its own commit or review conventions.
