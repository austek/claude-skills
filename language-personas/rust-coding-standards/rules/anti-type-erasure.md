---
title: Reach for Box<dyn Trait> Only When Types Genuinely Vary at Runtime
impact: MEDIUM
impactDescription: Trades away monomorphized static dispatch for heap allocation nobody needed
tags: [anti-pattern, trait-objects, generics, dispatch]
---

# Reach for Box<dyn Trait> Only When Types Genuinely Vary at Runtime [MEDIUM]

## Description
`Box<dyn Trait>` exists to hold values whose concrete type genuinely isn't known until runtime — a `Vec` mixing several different `Handler` implementations, a database backend chosen from a config file. Reaching for it out of habit when the concrete type actually is known at compile time (`Box::new((0..10).map(|x| x * 2))` returned as `Box<dyn Iterator<...>>` from a function with exactly one possible caller and one possible implementation) pays for something that isn't being used: heap allocation and a vtable indirection on every call, in exchange for flexibility nothing in the program actually exercises. `impl Trait` (in return position) or a plain generic parameter gets the same ergonomics with the concrete type resolved and inlined at compile time instead.

## Bad Example
```rust
// Only one concrete type is ever produced here — the erasure buys nothing.
fn doubled() -> Box<dyn Iterator<Item = i32>> {
    Box::new((0..10).map(|x| x * 2))
}
```

## Good Example
```rust
fn doubled() -> impl Iterator<Item = i32> {
    (0..10).map(|x| x * 2) // same call-site ergonomics, no heap allocation
}
```

## Notes
- `Box<dyn Trait>` is the right call, not a mistake, for a genuinely heterogeneous collection (`Vec<Box<dyn EventHandler>>` holding several distinct handler types), a type selected at runtime from configuration, or breaking a self-referential/recursive type that can't otherwise have a fixed size.
- A closed, known-in-advance set of variants — "this is always one of these three concrete kinds" — is frequently better modeled as an enum matched with `match` than as `Box<dyn Trait>` with runtime dispatch; the enum gets exhaustiveness checking the trait object can't offer.
- Return-position `impl Trait` in a trait definition (RPITIT, stable since Rust 1.75) covers a case that used to force `Box<dyn Trait>` as a workaround — a trait method returning `impl Display` no longer needs boxing just because the concrete return type varies by implementor, as long as callers don't need to name that type.

## References
- [anti-over-abstraction](anti-over-abstraction.md)
- [trait-dyn-vs-generic](trait-dyn-vs-generic.md)
- [mem-box-large-variant](mem-box-large-variant.md)
- [closure-static-vs-dyn](closure-static-vs-dyn.md)
