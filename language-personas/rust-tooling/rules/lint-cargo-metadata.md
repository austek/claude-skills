---
title: Enable clippy::cargo for Publishable Crates
impact: LOW
impactDescription: Catches packaging omissions before crates.io rejects or shames the publish
tags: [tooling, clippy, cargo, packaging, metadata]
---

# Enable clippy::cargo for Publishable Crates [LOW]

## Description
`clippy::cargo` is a lint group that inspects `Cargo.toml` rather than your Rust source — it flags a missing `description`, `license`, or `repository` field, a dependency pinned to a wildcard version, or a feature named in a way that fights Cargo's opt-in convention (`no-std` instead of `std`). None of this affects whether the code compiles or runs correctly; it only matters once the crate is meant to be published, at which point these are exactly the details a `crates.io` visitor or a downstream `cargo audit` run would notice missing. Turning the group on early means the metadata stays correct incrementally instead of becoming a last-minute checklist right before a release.

## Bad Example
```toml
# Cargo.toml — nothing here signals this is meant to be published.
[package]
name = "my-crate"
version = "0.1.0"

[dependencies]
serde = "*"  # any version — will drift silently and unpredictably
```

## Good Example
```toml
[package]
name = "my-crate"
version = "0.1.0"
edition = "2021"
description = "A short description of what this crate does"
license = "MIT OR Apache-2.0"
repository = "https://github.com/user/my-crate"

[dependencies]
serde = "1.0" # pinned to a real version range, not a wildcard

[lints.clippy]
cargo = "warn"
```

## Notes
- `cargo_common_metadata` is the single lint most worth enabling on its own — it's the one that catches a missing `description`/`license`/`repository`, the three fields every published crate needs.
- `multiple_crate_versions` flags when your dependency tree pulls in two different major versions of the same crate; it's often unavoidable in large trees and worth `allow`-ing per-project rather than treating as a hard failure.
- `negative_feature_names` and `redundant_feature_names` catch feature naming that fights Cargo convention — features should read as opt-in additions (`std`), not opt-outs (`no-std`), and shouldn't duplicate the crate's own name.
- For crates that will never be published (internal workspace members, application binaries), set `cargo = "allow"` rather than leaving the group enabled and ignoring its warnings — an ignored warning that never gets fixed is worse than no warning.

## References
- [doc-cargo-metadata](../../rust-coding-standards/rules/doc-cargo-metadata.md)
- [proj-workspace-deps](proj-workspace-deps.md)
- [lint-deny-correctness](lint-deny-correctness.md)
