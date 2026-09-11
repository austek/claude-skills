---
title: Use #[repr(transparent)] for FFI Newtypes
impact: MEDIUM
impactDescription: Guarantees a newtype's ABI matches its inner type exactly
tags: [type-safety, ffi, repr-transparent, newtype]
---

# Use #[repr(transparent)] for FFI Newtypes [MEDIUM]

## Description
A plain Rust struct's memory layout is unspecified by default — the compiler is free to reorder fields or add padding, which is fine for pure Rust code but breaks the moment the type crosses an FFI boundary expecting a specific C ABI layout. `#[repr(transparent)]` guarantees a single-field newtype has exactly the same size, alignment, and ABI as its inner type, so a `Handle(u64)` can be passed anywhere a raw `u64` is expected on the C side while still giving Rust callers type-safe wrapping. It applies only to single-field structs (a second field must be zero-sized, like `PhantomData`).

## Bad Example
```rust
// No layout guarantee -- may not match the inner u64's ABI.
struct Handle(u64);

extern "C" {
    fn process_handle(h: Handle); // may not work correctly
}
```

## Good Example
```rust
// Guaranteed identical layout to u64.
#[repr(transparent)]
struct Handle(u64);

extern "C" {
    fn process_handle(h: Handle); // safe: same ABI as u64
}
```

## Notes
- `size_of::<Handle>() == size_of::<u64>()` and `align_of::<Handle>() == align_of::<u64>()` are guaranteed, not incidental — that's the entire point of the attribute.
- A second field is allowed only if it's zero-sized (`PhantomData<T>`); it contributes nothing to layout, so the type stays ABI-identical to its one real field.
- Wrapping a `NonZeroU64` with `#[repr(transparent)]` keeps the niche optimization — `Option<Wrapper>` stays the same size as `u64`.
- Outside FFI, `#[repr(transparent)]` doesn't hurt on a pure-Rust newtype, but it's optional there — the layout guarantee only matters once another ABI is involved.

## References
- [type-newtype-ids](type-newtype-ids.md)
- [type-phantom-marker](type-phantom-marker.md)
- [api-newtype-safety](api-newtype-safety.md)
