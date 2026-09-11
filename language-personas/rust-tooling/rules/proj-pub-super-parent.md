---
title: Use pub(super) for Parent-Module-Only Visibility
impact: LOW
impactDescription: Confines a shared helper's visibility to exactly the sibling modules that need it
tags: [tooling, visibility, encapsulation, modules]
---

# Use pub(super) for Parent-Module-Only Visibility [LOW]

## Description
`pub(crate)` and plain private visibility are the two extremes most Rust code reaches for, but there's a useful middle option for a group of sibling modules that need to share something amongst themselves without exposing it to the rest of the crate: `pub(super)` makes an item visible only to its immediate parent module (and, by extension, that parent's other children). A `Token` type shared between a parser's `lexer` and `ast` submodules is a natural fit — the rest of the crate, including unrelated modules like `codegen`, has no legitimate reason to reach into parser internals, and `pub(super)` enforces that boundary at compile time instead of relying on convention.

## Bad Example
```rust
// src/parser/lexer.rs
pub fn internal_helper() {} // visible to the ENTIRE crate, not just parser/*

pub(crate) struct Token {} // also crate-wide, though only parser/* needs it
```

## Good Example
```rust
// src/parser/mod.rs
pub(super) struct Token {
    pub(super) kind: TokenKind,
}

pub(super) fn shared_helper() -> Token { /* ... */ }

// src/parser/lexer.rs
use super::{Token, shared_helper};

pub fn lex(input: &str) -> Vec<Token> {
    shared_helper();
    // ...
}
```

## Notes
- The scope is exactly the declaring item's parent module — from `src/parser/mod.rs`, `pub(super)` items are reachable in `lexer.rs` and `ast.rs` (parser's children) but completely invisible to `src/codegen.rs`, which sits outside the `parser` module tree entirely.
- Reach for `pub(super)` specifically for helper functions or shared types used by a module's own submodules but nowhere else in the crate — anything a wider set of modules legitimately needs belongs at `pub(crate)` instead.
- The same pattern extends to test helpers: a `pub(super) fn make_test_token()` defined inside a `mod tests` block stays visible to that module's sibling test modules without leaking into the crate's broader test infrastructure.
- Layering all four visibility levels deliberately — `pub` for the real public API, `pub(crate)` for crate-wide internals, `pub(super)` for a module group's shared helpers, private for everything else — communicates intended scope far more precisely than defaulting everything to `pub(crate)` out of convenience.

## References
- [proj-pub-crate-internal](proj-pub-crate-internal.md)
- [proj-pub-use-reexport](proj-pub-use-reexport.md)
- [proj-mod-by-feature](proj-mod-by-feature.md)
