---
title: Fill In Cargo.toml Metadata Before Publishing
impact: LOW
impactDescription: Improves crates.io discoverability and trust signals
tags: [documentation, cargo, publishing, metadata]
---

# Fill In Cargo.toml Metadata Before Publishing [LOW]

## Description
`[package]` metadata is what a stranger sees before they ever open the source: the one-line `description` on a search results page, the `license` that decides whether legal will let them depend on it, the `repository` link that lets them check if it's maintained. A `Cargo.toml` with only `name`, `version`, and `edition` publishes fine — `cargo publish` only strictly requires `description` and a license field — but it leaves crates.io with nothing to show except a bare name, which reads as either abandoned or untrustworthy to someone evaluating it for the first time.

## Bad Example
```toml
[package]
name = "retry-loop"
version = "0.1.0"
edition = "2021"
```

## Good Example
```toml
[package]
name = "retry-loop"
version = "0.1.0"
edition = "2021"
rust-version = "1.75"
description = "Configurable retry policies with exponential backoff"
license = "MIT OR Apache-2.0"
repository = "https://github.com/example/retry-loop"
documentation = "https://docs.rs/retry-loop"
readme = "README.md"
keywords = ["retry", "backoff", "resilience"]
categories = ["asynchronous"]
```

## Notes
- `description`, `license` (or `license-file`), and a valid `version` are the fields `cargo publish` actually enforces — everything past that is optional but strongly expected by anyone browsing crates.io.
- `keywords` caps out at five entries and rejects generic filler (`"rust"`, `"library"`) — pick terms someone would actually search for.
- Run `cargo package --list` before publishing to see exactly which files ship, and `cargo publish --dry-run` to catch a missing required field before it becomes a permanent version on the registry.

## References
- [doc-module-inner](doc-module-inner.md)
