---
title: Declare rust-version (MSRV) and Check It in CI
impact: MEDIUM
impactDescription: Replaces a cryptic downstream compile error with a clear "toolchain too old" message
tags: [tooling, cargo, msrv, ci, dependency-resolution]
---

# Declare rust-version (MSRV) and Check It in CI [MEDIUM]

## Description
Without a declared `package.rust-version`, a user on an old toolchain who hits an incompatibility gets whatever confusing type error or missing-feature message the actual incompatible code happens to produce, often many layers removed from the real cause. Setting `rust-version` in `Cargo.toml` gives Cargo enough information to produce a direct, actionable error — "this crate requires rustc 1.75 or newer" — the moment the toolchain is too old, instead of a cryptic failure deep inside a function nobody was looking at. It also feeds into dependency resolution: edition 2024's resolver (resolver = "3", the edition's default) is MSRV-aware, meaning it avoids pulling in a dependency version whose own declared `rust-version` exceeds yours, which prevents a routine `cargo update` from silently raising your crate's effective minimum toolchain out from under you.

## Bad Example
```toml
[package]
name = "my-crate"
version = "0.1.0"
edition = "2021"
# no rust-version declared — a routine dependency bump can silently raise
# the effective floor, and old-toolchain users get a confusing error
# instead of a clear one.
```

## Good Example
```toml
[package]
name = "my-crate"
version = "0.1.0"
edition = "2024"
rust-version = "1.79" # oldest toolchain this crate commits to supporting

[workspace]
resolver = "3" # default under edition 2024; makes resolution MSRV-aware
```

```yaml
# .github/workflows/msrv.yml
- uses: dtolnay/rust-toolchain@master
  with:
    toolchain: "1.79"
- run: cargo check --all-features
```

## Notes
- Pick the actual floor your real users need — check what distro-packaged Rust versions, embedded toolchains, or corporate environments with frozen dependencies are realistically still on — rather than guessing conservatively low or just matching whatever version happens to be installed locally.
- A lower MSRV isn't free: it widens the range of dependency versions Cargo is willing to select, which can mean landing on an older, possibly buggier release of a transitive dependency just to satisfy compatibility with a toolchain few if any users actually run.
- Treat raising the MSRV as a semver-relevant change for a published library, and call it out explicitly in the changelog — downstream crates with a lower MSRV of their own may break the moment they pull in the new version.
- The `cargo-msrv` tool automates finding the real floor by bisecting toolchain versions against `cargo check`, which is more reliable than guessing based on which language features the code happens to use.

## References
- [proj-workspace-deps](proj-workspace-deps.md)
- [lint-cargo-metadata](lint-cargo-metadata.md)
- [doc-cargo-metadata](../../rust-coding-standards/rules/doc-cargo-metadata.md)
