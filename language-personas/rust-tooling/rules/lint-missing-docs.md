---
title: Warn on missing_docs for Public API Surface
impact: MEDIUM
impactDescription: Forces every new public item to ship with a description, not just a signature
tags: [tooling, lints, documentation, missing-docs]
---

# Warn on missing_docs for Public API Surface [MEDIUM]

## Description
For a library, the public API's doc comments are effectively the product — a consumer reads `docs.rs`, not the source tree, to learn what a type or function does. The built-in `missing_docs` lint enforces that every `pub` item (structs, their public fields, functions, traits, and trait methods) carries a `///` comment, and it leaves private items alone entirely, so it never pressures you to document implementation details nobody outside the crate will ever see. Turning it on early, even at `warn`, means documentation coverage grows alongside the API instead of becoming a backlog that has to be paid down all at once before a release.

## Bad Example
```rust
#![warn(missing_docs)]

pub struct User {       // WARN: missing documentation for a struct
    pub name: String,   // WARN: missing documentation for a field
}

pub fn process() {}     // WARN: missing documentation for a function
```

## Good Example
```rust
#![warn(missing_docs)]

//! User management module.

/// Represents a registered user in the system.
pub struct User {
    /// The user's display name.
    pub name: String,
}

/// Processes pending user requests.
pub fn process() {}
```

## Notes
- The lint is scoped to `pub` items only — a private `struct Internal { .. }` never triggers a warning, so enabling `missing_docs` doesn't obligate you to comment implementation details.
- `#[allow(missing_docs)]` on a specific `pub mod internal { .. }` is the right tool for a deliberately undocumented internal-but-technically-public module (common in crates that expose internals for macro support), rather than disabling the lint crate-wide.
- For an existing codebase with a large undocumented surface, start at `warn`, fix items incrementally, then flip to `#![deny(missing_docs)]` once coverage is complete — going straight to `deny` on a legacy crate just blocks every unrelated PR until someone does a full documentation pass.
- Pair it with `#![warn(rustdoc::broken_intra_doc_links)]` — a doc comment that exists but links to a renamed or removed item is nearly as unhelpful as no doc comment at all, and that lint catches exactly that drift.

## References
- [doc-all-public](../../rust-coding-standards/rules/doc-all-public.md)
- [lint-unsafe-doc](lint-unsafe-doc.md)
- [test-doctest-examples](../../rust-testing/rules/test-doctest-examples.md)
