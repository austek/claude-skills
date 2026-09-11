---
title: Use a Blanket Impl to Cover an Entire Class of Types
impact: MEDIUM
impactDescription: Replaces per-type boilerplate with one impl over a bound
tags: [trait-design, blanket-impl, generics, coherence]
---

# Use a Blanket Impl to Cover an Entire Class of Types [MEDIUM]

## Description
A blanket impl — `impl<T: Bound> Trait for T` — extends every type satisfying `Bound` at once, instead of writing the same impl by hand for each concrete type. The standard library relies on this: `ToString` is blanket-implemented for every `T: Display`, so any `Display` type gets `.to_string()` automatically. The trade-off is coherence: because stable Rust has no specialization, a blanket impl permanently forecloses any more-specific impl for a type it already covers — that's a hard compiler error (E0119), not a warning — and adding a public blanket impl is a semver-relevant change since it can collide with impls downstream crates already provide.

## Bad Example
```rust
trait Describe {
    fn describe(&self) -> String;
}

// Manual impl per type -- tedious and doesn't scale.
impl Describe for i32 {
    fn describe(&self) -> String { format!("i32: {self}") }
}
impl Describe for f64 {
    fn describe(&self) -> String { format!("f64: {self}") }
}
```

## Good Example
```rust
use std::fmt;

trait Describe {
    fn describe(&self) -> String;
}

// One impl covers every T: Display, mirroring std's ToString.
impl<T: fmt::Display> Describe for T {
    fn describe(&self) -> String {
        format!("{self} ({})", std::any::type_name::<T>())
    }
}
```

## Notes
- Blanket impls live in the crate that owns the *trait*, not the type — that's what satisfies the orphan rule.
- You cannot also write `impl Describe for MyType` once the blanket impl exists; that overlap is a coherence error, since specialization isn't stable. Use a newtype for a per-type override instead.
- Treat adding a new public blanket impl as at least a minor semver bump, and a major one if it could plausibly collide with a downstream crate's own impl.
- Pair a blanket impl with a sealed trait when you want the automatic behavior but need to prevent external crates from implementing the trait themselves.

## References
- [api-extension-trait](api-extension-trait.md)
- [api-sealed-trait](api-sealed-trait.md)
- [trait-coherence-newtype](trait-coherence-newtype.md)
