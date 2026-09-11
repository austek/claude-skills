---
title: Hide Macro-Generated Helpers Behind a doc hidden __private Module
impact: MEDIUM
impactDescription: Keeps internals free to change without breaking semver
tags: [macros, api-design, semver]
---

# Hide Macro-Generated Helpers Behind a doc hidden __private Module [MEDIUM]

## Description
A macro's expanded code frequently needs to call something the crate provides internally — a formatting helper, a marker trait, a small utility function — and because the expansion happens at the caller's call site, that something has to be public enough for the caller's crate to actually resolve it. Making it an ordinary public item solves the resolution problem but creates a new one: it's now part of the crate's documented API, discoverable by anyone browsing the docs, and semver rules say removing or renaming it later counts as a breaking change even though it was only ever meant to support macro expansion. Wrapping those items in a `#[doc(hidden)] pub mod __private` threads the needle — they're technically public, so macro expansions can reach them, but they're absent from generated documentation and understood by convention to carry no stability guarantee. `serde`, `thiserror`, and most other derive-macro crates rely on exactly this pattern.

## Bad Example
```rust
// lib.rs — the helper leaks into the public API
pub fn __format_value(v: &dyn std::fmt::Debug) -> String {
    format!("{v:?}")
}

#[macro_export]
macro_rules! debug_print {
    ($val:expr) => {
        println!("{}", $crate::__format_value(&$val));
    };
}
// A user browsing the docs sees __format_value and may depend on it directly —
// removing it later is a semver-breaking change even though it was never intended as API.
```

## Good Example
```rust
// lib.rs
#[doc(hidden)]
pub mod __private {
    // Technically public (required for macro call sites), but hidden from
    // rendered docs and clearly marked as an unstable implementation detail
    pub use crate::helpers::format_value;
}

mod helpers {
    pub fn format_value(v: &dyn std::fmt::Debug) -> String {
        format!("{v:?}")
    }
}

#[macro_export]
macro_rules! debug_print {
    ($val:expr) => {
        // Reference through __private; never through a bare crate-root path
        println!("{}", $crate::__private::format_value(&$val));
    };
}
```

## Notes
- The name `__private` itself carries meaning across the ecosystem — the leading double underscore is a widely recognized signal that a module, while technically reachable, was never intended to be used directly, independent of any documentation.
- `#[doc(hidden)]` is what actually keeps the module out of rendered docs, so it shouldn't be treated as optional, and any reference to something inside it from within a macro's expansion should go through `$crate::__private::...` rather than a path that assumes the macro's own crate context.
- Since `__private` isn't covered by the crate's semver guarantees despite being technically public, it's worth stating that explicitly somewhere a downstream user might look — a CHANGELOG note or a line in the README — so someone who stumbles onto it while browsing source knows not to depend on it staying stable.

## References
- [macro-rules-hygiene](macro-rules-hygiene.md)
- [macro-proc-two-crate](macro-proc-two-crate.md)
