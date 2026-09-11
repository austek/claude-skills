---
name: rust-tooling
description: Rust development tooling configuration and best practices for clippy, rustc lints, and Cargo workspace/project structure. Use when setting up a Rust project, configuring lints, or editing Cargo.toml or workspace layout.
paths:
  - "**/*.rs"
  - "Cargo.toml"
---

# Rust Tooling

A guide to Rust development tooling, adapted from the leonardomso/rust-skills catalog (MIT licensed — see [NOTICE.md](NOTICE.md)). Configuration best practices for linting and Cargo project/workspace structure.

## Categories

### Project Structure [MEDIUM]
Cargo workspace and module layout conventions that scale with crate size.

| Rule | Description |
|------|-------------|
| [proj-bin-dir](rules/proj-bin-dir.md) | Put multiple binaries in src/bin/ |
| [proj-build-rs-minimal](rules/proj-build-rs-minimal.md) | Keep build.rs minimal, deterministic, and idempotent |
| [proj-feature-additive](rules/proj-feature-additive.md) | Design Cargo features as strictly additive |
| [proj-flat-small](rules/proj-flat-small.md) | Keep small projects flat |
| [proj-lib-main-split](rules/proj-lib-main-split.md) | Keep main.rs thin, put logic in lib.rs |
| [proj-mod-by-feature](rules/proj-mod-by-feature.md) | Organize modules by feature, not by type |
| [proj-mod-rs-dir](rules/proj-mod-rs-dir.md) | Choose mod.rs or adjacent-file style consistently |
| [proj-msrv-declare](rules/proj-msrv-declare.md) | Declare rust-version (MSRV) and check it in CI |
| [proj-prelude-module](rules/proj-prelude-module.md) | Provide a prelude module for common imports |
| [proj-pub-crate-internal](rules/proj-pub-crate-internal.md) | Use pub(crate) to mark internal-only APIs |
| [proj-pub-super-parent](rules/proj-pub-super-parent.md) | Use pub(super) for parent-module-only visibility |
| [proj-pub-use-reexport](rules/proj-pub-use-reexport.md) | Use pub use to flatten the public API |
| [proj-workspace-deps](rules/proj-workspace-deps.md) | Inherit dependency versions from the workspace |
| [proj-workspace-large](rules/proj-workspace-large.md) | Use a Cargo workspace for multi-crate projects |

### Linting (Clippy & rustc) [MEDIUM]
Configure clippy and rustc lint levels to catch bugs and style issues before review.

| Rule | Description |
|------|-------------|
| [lint-cargo-metadata](rules/lint-cargo-metadata.md) | Enable clippy::cargo for publishable crates |
| [lint-cfg-check](rules/lint-cfg-check.md) | Declare custom cfgs and enable unexpected_cfgs |
| [lint-clippy-nursery-selected](rules/lint-clippy-nursery-selected.md) | Cherry-Pick clippy::nursery lints instead of the whole group |
| [lint-deny-correctness](rules/lint-deny-correctness.md) | Deny clippy::correctness instead of warning |
| [lint-missing-docs](rules/lint-missing-docs.md) | Warn on missing_docs for public API surface |
| [lint-pedantic-selective](rules/lint-pedantic-selective.md) | Cherry-Pick clippy::pedantic lints instead of the whole group |
| [lint-rustfmt-check](rules/lint-rustfmt-check.md) | Enforce cargo fmt --check in CI |
| [lint-unsafe-doc](rules/lint-unsafe-doc.md) | Require SAFETY comments on unsafe blocks |
| [lint-warn-complexity](rules/lint-warn-complexity.md) | Enable clippy::complexity to flag unnecessarily convoluted code |
| [lint-warn-perf](rules/lint-warn-perf.md) | Enable clippy::perf to catch avoidable inefficiencies |
| [lint-warn-style](rules/lint-warn-style.md) | Enable clippy::style for idiomatic patterns |
| [lint-warn-suspicious](rules/lint-warn-suspicious.md) | Enable clippy::suspicious to catch likely bugs |
| [lint-workspace-lints](rules/lint-workspace-lints.md) | Configure lints once at the workspace level |

## Quick Reference

### Project Structure
```rust
// src/main.rs -- thin entry point
use my_app::{run, Config};

fn main() -> anyhow::Result<()> {
    let config = Config::from_env()?;
    run(config)
}

// src/lib.rs -- the actual application logic, reachable from tests/
pub mod config;
pub mod database;

pub use config::Config;
```

### Linting (Clippy & rustc)
```toml
# Root Cargo.toml
[workspace.lints.rust]
unsafe_code = "deny"

[workspace.lints.clippy]
unwrap_used = "deny"
expect_used = "warn"

# crate-a/Cargo.toml
[lints]
workspace = true
```

## See Also

- [rust-coding-standards](../rust-coding-standards/SKILL.md) - General Rust coding standards and best practices
- [rust-testing](../rust-testing/SKILL.md) - Test-writing best practices for Rust
- [NOTICE](NOTICE.md) - MIT attribution for content adapted from leonardomso/rust-skills
