---
title: Use PhantomData to Express Type Relationships at Zero Cost
impact: MEDIUM
impactDescription: Encodes type-level state and variance with no runtime representation
tags: [type-safety, phantom-data, generics, zero-cost]
---

# Use PhantomData to Express Type Relationships at Zero Cost [MEDIUM]

## Description
A generic parameter that doesn't appear in any field is a compile error — Rust requires every type parameter to be "used." `PhantomData<T>` satisfies that requirement without storing an actual `T`: it's zero-sized, so it costs nothing at runtime, while still telling the compiler (and readers) that the type is conceptually associated with `T` for ownership, borrowing, variance, or purely for making otherwise-identical types distinct. The alternative — an `Option<T>` field nobody ever populates — wastes memory and forces an unnecessary `T: Default` or similar bound.

## Bad Example
```rust
// Error: parameter `T` is never used.
struct Handle<T> {
    id: u64,
}
```

## Good Example
```rust
use std::marker::PhantomData;

struct Handle<T> {
    id: u64,
    _marker: PhantomData<T>, // zero-size, tells the compiler about T
}

struct User;
struct Order;

fn process_user(h: Handle<User>) { /* ... */ }

let user_handle = Handle::<User> { id: 1, _marker: PhantomData };
let order_handle = Handle::<Order> { id: 2, _marker: PhantomData };

process_user(user_handle);   // OK
// process_user(order_handle); // compile error: Handle<Order> != Handle<User>
```

## Notes
- `PhantomData<T>` is covariant in `T`; `PhantomData<fn(T)>` is contravariant; `PhantomData<fn(T) -> T>` is invariant — pick the marker shape that matches the variance the type actually needs.
- `PhantomData<&'a T>` communicates a borrow of `T` for lifetime `'a` when the real borrow is hidden behind a raw pointer, so the borrow checker still tracks it correctly.
- Typestate patterns (`Door<Locked>` vs `Door<Unlocked>`) use a zero-sized marker type plus `PhantomData` to make illegal method calls a compile error rather than a runtime check.
- `#[repr(transparent)]` combined with `PhantomData` keeps a type's memory layout identical to its single real field, since the marker contributes no size or alignment.

## References
- [api-typestate](api-typestate.md)
- [api-newtype-safety](api-newtype-safety.md)
- [type-newtype-ids](type-newtype-ids.md)
