---
title: Don't Narrow a Function Parameter to &String
impact: LOW
impactDescription: Rejects string literals and &str values a caller would otherwise pass directly
tags: [anti-pattern, api-design, strings, code-smell]
---

# Don't Narrow a Function Parameter to &String [LOW]

## Description
`&String` only accepts a reference to a heap-allocated `String` — a caller holding a `&str`, a string literal, or a slice of one has to allocate a whole new `String` just to satisfy the signature, even though the function itself never uses anything a plain `&str` couldn't provide. This is a narrower special case of a more general habit — writing a parameter type that matches whatever the author happened to have on hand while writing the function, rather than the loosest type the function body actually requires. `&String` coerces to `&str` automatically via `Deref`, so switching the parameter costs nothing at existing call sites while opening the function to literals and borrowed strings that previously needed an unnecessary allocation to call it at all.

## Bad Example
```rust
fn greet(name: &String) {
    println!("Hello, {name}");
}

greet(&"Alice".to_string()); // forced to allocate just to call this
```

## Good Example
```rust
fn greet(name: &str) {
    println!("Hello, {name}");
}

greet("Alice");        // literal, no allocation
greet(&owned_string);  // &String -> &str via Deref, still works
```

## Notes
- `clippy::ptr_arg` flags this mechanically for `&String`, `&Vec<T>`, and `&PathBuf` alike — worth enabling as a lint rather than relying on catching it in review every time.
- The one legitimate reason to keep `&String` is needing a method that genuinely only exists on `String` (`.capacity()`, for instance) — that's rare enough to treat as the exception, not something to default to.
- `impl AsRef<str>` widens the parameter even further than `&str`, accepting anything that can cheaply produce a string view — reach for it specifically when a function needs to work equally well with owned and borrowed callers, not as a default replacement for every `&str` parameter.

## References
- [anti-vec-for-slice](anti-vec-for-slice.md)
- [own-slice-over-vec](own-slice-over-vec.md)
- [api-impl-asref](api-impl-asref.md)
