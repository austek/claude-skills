---
title: Derive the Crate-Root Doc Comment From the README
impact: LOW
impactDescription: Removes the drift between README.md, docs.rs, and the crates.io landing page
tags: [documentation, readme, rustdoc, cargo]
---

# Derive the Crate-Root Doc Comment From the README [LOW]

## Description
A `README.md` and a hand-written `//!` block at the top of `lib.rs` describe the same crate to two different audiences — GitHub/crates.io readers and docs.rs readers — and keeping both updated by hand means they will eventually say different things, usually because one got edited during a release and the other didn't. `#![doc = include_str!("../README.md")]` collapses the two into one file: the README becomes the crate's front page on docs.rs as well as on GitHub, and whichever one someone remembers to update is the one that matters. This works because attribute values can take a macro expansion in that position — a general capability (`extended_key_value_attributes`) that stabilized in Rust 1.54, not something specific to `doc`.

## Bad Example
```rust
// src/lib.rs — a second, independent description that will drift
//! Utilities for retrying operations with backoff.
//!
//! See the examples in the repository README for usage.

pub fn retry() { /* ... */ }
```

## Good Example
```rust
// src/lib.rs
#![doc = include_str!("../README.md")]

pub fn retry() { /* ... */ }
```
```toml
# Cargo.toml
[package]
readme = "README.md"
documentation = "https://docs.rs/retry-loop"
```

## Notes
- rustdoc tries to compile every fenced code block in the included README as a doctest — annotate anything that isn't runnable Rust (`bash`, `toml`, or `rust,no_run` / `rust,ignore` for Rust snippets that shouldn't execute).
- Set `readme = "README.md"` in `Cargo.toml` too, so crates.io's own landing page (which reads that field directly, not the compiled doc output) shows the same content.
- This only helps when the README's structure already works as a docs.rs front page — a README built entirely around badges and a contribution guide may need its own separate crate-level summary instead.

## References
- [doc-module-inner](doc-module-inner.md)
- [doc-cargo-metadata](doc-cargo-metadata.md)
- [doc-all-public](doc-all-public.md)
