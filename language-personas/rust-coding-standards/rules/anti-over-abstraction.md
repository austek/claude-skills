---
title: Don't Generalize Before a Second Concrete Use Case Exists
impact: LOW
impactDescription: Trades real compile time and readability for flexibility nobody has used yet
tags: [anti-pattern, generics, api-design, yagni]
---

# Don't Generalize Before a Second Concrete Use Case Exists [LOW]

## Description
Generics and trait abstractions are not free — every additional type parameter is compile time, binary size from monomorphization, and a reader having to hold one more layer of indirection in their head before reaching the actual logic. Paying that cost preemptively, for a trait with exactly one implementation or a function generalized over types nobody has actually needed to vary, buys speculative flexibility that may never be exercised while definitely costing something today. A concrete function or struct is easier to read, faster to compile, and just as easy to generalize later, once a second real use case shows up and makes clear what the abstraction should actually look like.

## Bad Example
```rust
// One caller, one concrete need — the generality has no current use.
fn add<T, U, R>(a: T, b: U) -> R
where
    T: Into<R>,
    U: Into<R>,
    R: std::ops::Add<Output = R>,
{
    a.into() + b.into()
}
```

## Good Example
```rust
// Concrete, obvious, and just as easy to generalize later if a second
// caller actually needs a different numeric type.
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

## Notes
- A useful trigger for when to generalize: two or three concrete implementations already exist and clearly share behavior — abstracting at that point is extracting a pattern that's proven itself, not guessing at one that might exist.
- Watch for the specific tells: a trait with only one `impl`, a type parameter list that's grown past what a reader can track (`T, U, V, W`), a marker trait with no methods, or a `where` clause stacking four-plus bounds on a single parameter.
- Internal, non-public functions almost never need the abstraction — reserve generic parameters for genuine polymorphism needs or for a public API where callers legitimately bring different concrete types.

## References
- [type-generic-bounds](type-generic-bounds.md)
- [api-sealed-trait](api-sealed-trait.md)
- [anti-type-erasure](anti-type-erasure.md)
