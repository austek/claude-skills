---
title: Use a Cargo Workspace for Multi-Crate Projects
impact: MEDIUM
impactDescription: Gives related crates one shared lock file and build cache instead of N independent ones
tags: [tooling, cargo, workspace, project-structure]
---

# Use a Cargo Workspace for Multi-Crate Projects [MEDIUM]

## Description
Splitting a large system into `my-app-core`, `my-app-cli`, `my-app-server`, and `my-app-common` as entirely separate repositories means each one gets its own `Cargo.lock`, its own build cache, and no built-in mechanism keeping their shared dependencies in sync — a fix in `core` requires publishing it (or pointing at a git commit) before `cli` or `server` can pick it up, which makes cross-crate development slow and error-prone. A Cargo workspace keeps the same crate boundaries — each member is still its own independently buildable, independently publishable crate — but puts them under one root `Cargo.toml`, sharing a single `Cargo.lock` and build cache, so a local change in `core` is immediately visible to `cli` and `server` without any publish step, and `cargo build --workspace` builds everything with maximal shared incremental compilation.

## Bad Example
```
my-app-core/      # separate repo, separate Cargo.lock
my-app-cli/        # separate repo, separate Cargo.lock
my-app-server/     # separate repo, separate Cargo.lock
# dependency versions drift independently; cross-crate changes require
# publishing or git-pinning before downstream crates can pick them up
```

## Good Example
```toml
# Root Cargo.toml — virtual workspace, no [package] of its own
[workspace]
resolver = "3" # default for edition 2024; use "2" for 2021
members = ["crates/core", "crates/cli", "crates/server"]

[workspace.dependencies]
tokio = { version = "1.0", features = ["full"] }

[workspace.lints.rust]
unsafe_code = "forbid"
```
```toml
# crates/core/Cargo.toml
[package]
name = "my-app-core"
version = "0.1.0"

[dependencies]
tokio = { workspace = true }

[lints]
workspace = true
```

## Notes
- A single binary or library crate doesn't need a workspace at all — reach for one once there are genuinely multiple related crates: a library plus a CLI plus a server, a set of internal shared libraries, or a plugin-style architecture where several crates need to build and evolve together.
- `cargo build --workspace`, `cargo test --workspace`, and `cargo check --workspace` operate across every member at once; `cargo build -p my-app-core` (or `cargo run -p my-app-cli`) targets a single member when you don't need the whole tree.
- A "virtual" workspace root — one with a `[workspace]` table but no `[package]` section of its own — is the common shape when the root directory isn't itself a crate, just a container for the members listed under it.
- `[workspace.lints.*]` and `[workspace.dependencies]` both compound the workspace's core benefit: not just a shared lock file, but shared lint policy and shared dependency versions that every member inherits with one `workspace = true` line instead of redeclaring them.

## References
- [proj-workspace-deps](proj-workspace-deps.md)
- [proj-bin-dir](proj-bin-dir.md)
- [proj-lib-main-split](proj-lib-main-split.md)
