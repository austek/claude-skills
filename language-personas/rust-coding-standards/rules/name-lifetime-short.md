---
title: Keep Lifetime Names Short and Conventional
impact: LOW
impactDescription: Keeps generic signatures scannable instead of cluttered
tags: [naming, style, lifetimes, generics]
---

# Keep Lifetime Names Short and Conventional [LOW]

## Description
Lifetime parameters show up constantly in Rust function signatures, so their names need to be short enough not to dominate the line. `'a`, `'b`, `'c` are the default choice for a generic lifetime with no special meaning, exactly the way `T`, `U` work for type parameters. When a lifetime has a specific role worth naming, the convention favors a short domain word rather than a long descriptive phrase — `'de` for deserialization (as in serde's `Deserialize<'de>`), `'src` for a source buffer a parser borrows from — because the goal is a hint, not a sentence.

## Bad Example
```rust
// The verbose names add noise without adding information.
struct TokenStream<'source_buffer_lifetime> {
    source: &'source_buffer_lifetime str,
}

fn longest<'left_input_lifetime, 'right_input_lifetime>(
    a: &'left_input_lifetime str,
    b: &'right_input_lifetime str,
) -> &'left_input_lifetime str { ... }
```

## Good Example
```rust
struct TokenStream<'src> {
    source: &'src str,
}

fn longest<'a>(a: &'a str, b: &'a str) -> &'a str { ... }
```

## Notes
- Prefer eliding the lifetime entirely when the compiler can infer it — `fn name(&self) -> &str` needs no explicit `'a` at all; see `own-lifetime-elision`.
- `'static` is the one lifetime name that's fixed by the language itself, not a convention — it means "valid for the whole program," not just "a long-lived generic parameter."
- Serde's `'de` is worth learning specifically: any type implementing `Deserialize<'de>` that borrows from its input (`#[serde(borrow)]`) should use exactly that name, since it's what the derive macro and surrounding ecosystem expect to see.

## References
- [name-type-param-single](name-type-param-single.md)
