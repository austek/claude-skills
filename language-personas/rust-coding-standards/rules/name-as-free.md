---
title: Reserve the as_ Prefix for Free Reference Conversions
impact: MEDIUM
impactDescription: Lets callers trust as_ methods are O(1) with no surprise allocation
tags: [naming, api-design, conversions, borrowing]
---

# Reserve the as_ Prefix for Free Reference Conversions [MEDIUM]

## Description
Method name prefixes carry a cost contract in Rust: `as_` promises a reference-to-reference reinterpretation with no allocation and no computation — `&Self -> &U`. When a method is named `as_something` but secretly allocates or does real work, callers who assumed it was free will call it in loops or hot paths without a second thought, and the mismatch between name and cost becomes a silent performance trap. The convention only holds value if every `as_` method actually is free; std enforces this rigorously across `String::as_bytes`, `Vec::as_slice`, and `PathBuf::as_path`, all of which just reinterpret existing memory.

## Bad Example
```rust
struct DisplayName(String);

impl DisplayName {
    // Named like a free reference cast, but format! allocates.
    fn as_upper(&self) -> String {
        self.0.to_uppercase()
    }
}
```

## Good Example
```rust
struct DisplayName(String);

impl DisplayName {
    // Genuinely free: returns a reference into existing data.
    fn as_str(&self) -> &str {
        &self.0
    }

    // Allocation is honest here: the to_ prefix says "this costs something".
    fn to_upper(&self) -> String {
        self.0.to_uppercase()
    }
}
```

## Notes
- Pair this with `to_` (allocates/computes, `&Self -> U`) and `into_` (consumes `self`, usually cheap) to cover the full conversion-naming triad — see `name-to-expensive` and `name-into-ownership`.
- A quick self-test before naming a method `as_x`: does it call `.clone()`, `format!`, `String::from`, or any allocator-touching function? If yes, it isn't `as_`.
- `AsRef`/`AsMut` trait implementations should follow the same rule — a type implementing `AsRef<str>` that secretly allocates on every call breaks the trait's implicit contract just as badly as a bespoke method would.

## References
- [name-to-expensive](name-to-expensive.md)
- [name-into-ownership](name-into-ownership.md)
