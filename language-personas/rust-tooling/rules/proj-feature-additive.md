---
title: Design Cargo Features as Strictly Additive
impact: HIGH
impactDescription: Prevents feature unification from silently breaking any crate that didn't ask for the feature
tags: [tooling, cargo, features, dependency-graph]
---

# Design Cargo Features as Strictly Additive [HIGH]

## Description
Cargo's feature system unifies across the whole dependency graph: if any crate anywhere in the build enables a feature on a shared dependency, every other crate depending on it gets that feature too, whether it asked for it or not. That single fact has a hard consequence — a feature that changes or removes existing behavior will break every consumer relying on the baseline the instant some unrelated third dependency turns it on, and the breakage has nothing to do with anything the broken crate itself did. The only safe design is for a feature to ever add — a new trait impl, an optional integration, extra capability — never to subtract or alter what's already there by default; a `no_std` feature that removes `std` support when enabled is a textbook violation, precisely inverted from how the flag should work.

## Bad Example
```toml
[features]
# Enabling "no_std" REMOVES behavior — a non-additive feature.
no_std = []
```
```rust
#[cfg(not(feature = "no_std"))]
use std::vec::Vec;
#[cfg(feature = "no_std")]
use alloc::vec::Vec;
```

## Good Example
```toml
[features]
default = ["std"]
std = []          # ADDS std support; the crate's baseline is no_std

serde = ["dep:serde"]  # purely additive optional integration
```
```rust
#![cfg_attr(not(feature = "std"), no_std)]

#[cfg(feature = "std")]
use std::vec::Vec;
#[cfg(not(feature = "std"))]
use alloc::vec::Vec;
```

## Notes
- If a crate is meant to support `no_std` environments, that has to be the unconditional baseline, with `std` as the opt-in addition — never the other way around, since inverting it means enabling `std` (the common case for most consumers) would be the thing silently subtracted when unified against a `no_std`-only build elsewhere in the graph.
- Cargo has no built-in way to enforce that two features are mutually exclusive; if your crate genuinely can't support two backends enabled simultaneously, detect that combination and fail loudly with `compile_error!("feature \"backend-a\" and \"backend-b\" are mutually exclusive")` rather than letting it silently pick one or produce nonsensical behavior.
- Prefix optional dependencies with `dep:` in the feature definition (`serde = ["dep:serde"]`) so the dependency's crate name doesn't also become an implicit feature name of its own — this keeps your feature namespace under your own control.
- Before adding a feature, ask what happens if some unrelated dependency three levels down in the graph turns it on for you — if the answer involves anything changing for consumers who never opted in, the feature isn't additive yet.

## References
- [api-serde-optional](../../rust-coding-standards/rules/api-serde-optional.md)
- [proj-workspace-deps](proj-workspace-deps.md)
- [lint-cfg-check](lint-cfg-check.md)
