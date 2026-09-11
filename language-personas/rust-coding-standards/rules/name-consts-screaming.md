---
title: Name Constants and Statics in SCREAMING_SNAKE_CASE
impact: LOW
impactDescription: Matches rustc's non_upper_case_globals lint, avoids a build warning
tags: [naming, style, constants, statics]
---

# Name Constants and Statics in SCREAMING_SNAKE_CASE [LOW]

## Description
`const` and `static` items are resolved entirely at compile time and, for statics, live for the whole program — they're a different kind of thing from a stack-local `let` binding, and their casing should say so at a glance. `SCREAMING_SNAKE_CASE` is the marker: seeing `MAX_RETRIES` versus `max_retries` tells a reader immediately whether they're looking at a compile-time constant or a value computed at runtime, without needing to trace where it's declared. rustc backs this with the `non_upper_case_globals` lint, which warns by default on any `const`/`static` item that doesn't follow the convention.

## Bad Example
```rust
const maxRetries: u32 = 5;          // warning: should be SCREAMING_SNAKE_CASE
static requestTimeout: u64 = 30;    // warning
```

## Good Example
```rust
const MAX_RETRIES: u32 = 5;
static REQUEST_TIMEOUT_SECS: u64 = 30;

impl ConnectionPool {
    // Associated constants follow the same casing.
    const DEFAULT_SIZE: usize = 16;
}
```

## Notes
- The casing rule applies equally to trait-level associated constants (`trait Limits { const MAX: usize; }`) and to constants scoped inside a module or impl block.
- A `static` that must be mutated at runtime needs interior mutability or an atomic type (`AtomicU64`, `OnceLock`) — the naming convention doesn't change, only the declared type does.
- Don't conflate this with `name-types-camel`: a `const` binding to a value is screaming-snake regardless of whether its type is a struct — only the item's own kind (value vs. type) decides the casing.

## References
- [name-funcs-snake](name-funcs-snake.md)
- [name-types-camel](name-types-camel.md)
