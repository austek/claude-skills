---
title: Inherit Dependency Versions from the Workspace
impact: HIGH
impactDescription: Eliminates version drift across crates that otherwise bloats binaries and slows compilation
tags: [tooling, cargo, workspace, dependencies]
---

# Inherit Dependency Versions from the Workspace [HIGH]

## Description
In a workspace where each member crate declares its own dependency versions independently, drift is close to inevitable — one crate ends up on `serde 1.0.150`, another on `1.0.188`, with nobody having decided that on purpose. Beyond the maintenance headache, actual different-major-or-incompatible versions of the same crate in one dependency tree mean the final binary links both, compile times grow because both get built, and behavior can subtly differ between the two call sites depending on which version each one got. Workspace dependency inheritance — stable since Rust 1.64 — fixes this at the source: declare a dependency once under `[workspace.dependencies]` in the root `Cargo.toml`, and each member crate opts in with `name.workspace = true`, guaranteeing every crate that inherits it is genuinely on the identical version rather than merely a compatible-looking one.

## Bad Example
```toml
# crate-a/Cargo.toml
[dependencies]
serde = "1.0.150"

# crate-b/Cargo.toml
[dependencies]
serde = "1.0.188"  # different version, decided independently
```

## Good Example
```toml
# Root Cargo.toml
[workspace]
members = ["crate-a", "crate-b"]

[workspace.dependencies]
serde = { version = "1.0", features = ["derive"] }

# crate-a/Cargo.toml
[dependencies]
serde.workspace = true

# crate-b/Cargo.toml
[dependencies]
serde.workspace = true
```

## Notes
- A member crate can still add its own features on top of the inherited version: `tokio = { workspace = true, features = ["net", "io-util"] }` gets both the workspace's baseline features and the extras listed locally, without needing to redeclare the version.
- `optional = true` has to be set at the member-crate level, not on the workspace declaration — `[workspace.dependencies]` only fixes the version and shared features; whether a given crate treats the dependency as optional is still a per-crate decision.
- The same inheritance mechanism works for internal path dependencies (`my-core = { path = "crates/core" }` declared once under `[workspace.dependencies]`), which is useful in a workspace with many internal crates depending on each other.
- `[workspace.package]` extends the same idea to shared package metadata (`version`, `edition`, `license`) — member crates pick it up with `version.workspace = true` the same way they inherit a dependency, keeping metadata as consistent across the workspace as dependency versions.

## References
- [proj-lib-main-split](proj-lib-main-split.md)
- [api-serde-optional](../../rust-coding-standards/rules/api-serde-optional.md)
- [lint-deny-correctness](lint-deny-correctness.md)
