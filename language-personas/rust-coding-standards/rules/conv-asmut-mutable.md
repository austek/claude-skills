---
title: Accept impl AsMut for Flexible Mutable Inputs
impact: LOW
impactDescription: Widens a function to accept Vec, arrays, and slices with no conversion cost
tags: [conversions, as-mut, generics, api-design]
---

# Accept impl AsMut for Flexible Mutable Inputs [LOW]

## Description
`AsMut<T>` is the mutable mirror of `AsRef<T>`. A function that takes `&mut Vec<u8>` rejects a caller holding a `&mut [u8]` or a `&mut [u8; N]` array, even though the body only ever needs mutable byte access. Accepting `impl AsMut<[u8]>` instead widens the function to all three input shapes with zero conversion cost and no change to the implementation. This is worth doing for genuinely generic write targets — not every `&mut T` parameter benefits from it, and forcing it onto a function that will only ever see one concrete type just adds compile time and a layer of indirection for no payoff.

## Bad Example
```rust
// Only accepts &mut Vec<u8>; arrays and slices are excluded entirely.
fn fill_zeros(buf: &mut Vec<u8>) {
    for b in buf.iter_mut() {
        *b = 0;
    }
}

// let mut arr = [1u8, 2, 3];
// fill_zeros(&mut arr); // compile error
```

## Good Example
```rust
// Accepts Vec<u8>, [u8; N], and &mut [u8] -- anything that lends &mut [u8].
fn fill_zeros(mut buf: impl AsMut<[u8]>) {
    for b in buf.as_mut().iter_mut() {
        *b = 0;
    }
}

let mut arr_buf = [1u8, 2, 3, 4];
fill_zeros(&mut arr_buf); // now compiles
```

## Notes
- If a function only ever receives `&mut SomeConcreteType` in practice, keep the concrete parameter — the generic version adds cognitive overhead without a real caller to benefit.
- Avoid stacking `AsMut` and `AsRef` bounds on the same parameter unless the API genuinely needs both read and write access.
- Don't reach for `AsMut` as a stand-in for a trait that should capture actual domain semantics — name the abstraction for what it means, not just how it converts.
- Callers can pass a slice directly via `.as_mut()` when they already have one, so the generic parameter doesn't force an owned type on anyone.

## References
- [api-impl-asref](api-impl-asref.md)
- [own-slice-over-vec](own-slice-over-vec.md)
