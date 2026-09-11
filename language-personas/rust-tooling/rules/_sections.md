# Section Definitions

## Project Structure (proj)
**Impact:** MEDIUM

Cargo workspace and module layout conventions that scale with crate size.
Covers the lib/main split, module-per-feature organization, visibility scoping, and workspace dependency management.

**Rules:**
- `proj-bin-dir` - Put multiple binaries in src/bin/
- `proj-build-rs-minimal` - Keep build.rs minimal, deterministic, and idempotent
- `proj-feature-additive` - Design Cargo features as strictly additive
- `proj-flat-small` - Keep small projects flat
- `proj-lib-main-split` - Keep main.rs thin, put logic in lib.rs
- `proj-mod-by-feature` - Organize modules by feature, not by type
- `proj-mod-rs-dir` - Choose mod.rs or adjacent-file style consistently
- `proj-msrv-declare` - Declare rust-version (MSRV) and check it in CI
- `proj-prelude-module` - Provide a prelude module for common imports
- `proj-pub-crate-internal` - Use pub(crate) to mark internal-only APIs
- `proj-pub-super-parent` - Use pub(super) for parent-module-only visibility
- `proj-pub-use-reexport` - Use pub use to flatten the public API
- `proj-workspace-deps` - Inherit dependency versions from the workspace
- `proj-workspace-large` - Use a Cargo workspace for multi-crate projects

## Linting (Clippy & rustc) (lint)
**Impact:** MEDIUM

Configure clippy and rustc lint levels to catch bugs and style issues before review.
Covers workspace-wide lint tables, pedantic/nursery opt-in, deny-by-default correctness lints, and CI lint checks.

**Rules:**
- `lint-cargo-metadata` - Enable clippy::cargo for publishable crates
- `lint-cfg-check` - Declare custom cfgs and enable unexpected_cfgs
- `lint-clippy-nursery-selected` - Cherry-Pick clippy::nursery lints instead of the whole group
- `lint-deny-correctness` - Deny clippy::correctness instead of warning
- `lint-missing-docs` - Warn on missing_docs for public API surface
- `lint-pedantic-selective` - Cherry-Pick clippy::pedantic lints instead of the whole group
- `lint-rustfmt-check` - Enforce cargo fmt --check in CI
- `lint-unsafe-doc` - Require SAFETY comments on unsafe blocks
- `lint-warn-complexity` - Enable clippy::complexity to flag unnecessarily convoluted code
- `lint-warn-perf` - Enable clippy::perf to catch avoidable inefficiencies
- `lint-warn-style` - Enable clippy::style for idiomatic patterns
- `lint-warn-suspicious` - Enable clippy::suspicious to catch likely bugs
- `lint-workspace-lints` - Configure lints once at the workspace level
