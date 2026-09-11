---
title: Use pub(crate) to Mark Internal-Only APIs
impact: MEDIUM
impactDescription: Shrinks the surface a breaking change can actually break, from "any downstream user" to "this crate only"
tags: [tooling, visibility, encapsulation, api-design]
---

# Use pub(crate) to Mark Internal-Only APIs [MEDIUM]

## Description
Marking every item `pub` by default might seem harmless, but each one becomes part of a promise to external users the moment the crate is published — changing or removing it later is a breaking change, whether or not it was ever meant to be consumer-facing. `pub(crate)` draws a real boundary: the item is visible and usable anywhere inside the crate, including from sibling modules, but invisible outside it, so it can be refactored, renamed, or removed freely without touching semver. Reaching for `pub(crate)` (or leaving an item fully private when only its own module needs it) as the default, and promoting to `pub` only for what's deliberately part of the public contract, keeps that contract small and the internals free to evolve.

## Bad Example
```rust
// Everything pub — internal implementation details become part of the
// crate's semver contract whether that was intended or not.
pub mod internal {
    pub struct InternalState {
        pub buffer: Vec<u8>,
        pub dirty: bool,
    }
}

pub struct Widget {
    pub state: internal::InternalState, // now externally visible and touchable
}
```

## Good Example
```rust
pub(crate) mod internal {
    pub(crate) struct InternalState {
        pub(crate) buffer: Vec<u8>,
        pub(crate) dirty: bool,
    }
}

pub struct Widget {
    state: internal::InternalState, // private field — implementation detail
}

impl Widget {
    pub fn new() -> Self {
        Self { state: internal::InternalState { buffer: Vec::new(), dirty: false } }
    }
}
```

## Notes
- `pub(crate)` sits between fully private (visible only in the declaring module and its children) and fully `pub` (visible to any external crate) — it's the right choice for a helper that several modules within your own crate need to share but that consumers of the crate have no business touching.
- A common pattern is a private (or `pub(crate)`) internal module whose types stay hidden behind a genuinely public API module — `mod internal;` alongside `pub mod api;`, where `api` is the only thing an external user ever sees or depends on.
- For test-only access to internals that shouldn't otherwise be public, a `#[cfg(test)] pub(crate)` accessor or a `#[doc(hidden)] pub mod __test_helpers` keeps that access scoped to tests without widening the crate's real public surface.
- The payoff shows up specifically during refactoring: a change to a `pub(crate)` type only needs `cargo build` across your own crate to confirm nothing broke, whereas the same change to a `pub` type is a semver-breaking release regardless of how internal it felt at the time.

## References
- [proj-pub-super-parent](proj-pub-super-parent.md)
- [proj-pub-use-reexport](proj-pub-use-reexport.md)
- [api-non-exhaustive](../../rust-coding-standards/rules/api-non-exhaustive.md)
