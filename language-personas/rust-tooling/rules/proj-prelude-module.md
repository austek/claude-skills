---
title: Provide a Prelude Module for Common Imports
impact: LOW
impactDescription: Collapses a wall of individual imports into one glob import
tags: [tooling, project-structure, api-design, prelude]
---

# Provide a Prelude Module for Common Imports [LOW]

## Description
A crate with a moderately large public surface forces every consumer to write out individual `use` statements for each type they need, which is repetitive across nearly every file that touches the crate. A `prelude` module — following the same pattern as `std::prelude`, which is implicitly glob-imported into every Rust file — collects the handful of types and traits almost every user needs into one place, so a consumer writes a single `use my_crate::prelude::*;` instead of five or six separate lines. It's a small ergonomic win per file, but it compounds across a codebase that depends on the crate heavily, and it gives the crate author one clearly-labeled place to declare "these are the items we consider core."

## Bad Example
```rust
// Consumers write this in every file that touches the crate.
use my_crate::Client;
use my_crate::Config;
use my_crate::Error;
use my_crate::traits::Handler;
use my_crate::types::Method;
```

## Good Example
```rust
// src/lib.rs
pub mod prelude {
    pub use crate::{Client, Config, Error};
    pub use crate::traits::Handler;
    pub use crate::types::Method;
}
```
```rust
// consumer code
use my_crate::prelude::*;
```

## Notes
- Be conservative about what goes in — a prelude is meant for the small set of items nearly every consumer needs, not a dumping ground for anything `pub`; rarely-used types and internal helpers dilute the point of having a curated list at all.
- Watch for name collisions with common std or third-party names — `Error` is the classic offender, since a crate's `prelude::Error` glob-imported alongside another crate's own `Error` type produces an ambiguity the compiler will reject.
- Treat the prelude's contents as part of the crate's stability contract: removing an item from it is a breaking change for anyone who glob-imported it, even though adding new items is safe (this is the same additive-only discipline that governs Cargo features).
- Document what the prelude re-exports directly in its module doc comment — a short bullet list naming the key items — so a reader doesn't have to open the source to find out what a `use my_crate::prelude::*;` actually pulled in.

## References
- [proj-pub-use-reexport](proj-pub-use-reexport.md)
- [api-extension-trait](../../rust-coding-standards/rules/api-extension-trait.md)
- [doc-module-inner](../../rust-coding-standards/rules/doc-module-inner.md)
