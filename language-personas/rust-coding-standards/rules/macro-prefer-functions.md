---
title: Reach for a Macro Only When a Function Cannot Express It
impact: HIGH
impactDescription: Keeps type inference, IDE navigation, and error messages intact for ordinary logic
tags: [macros, api-design, simplicity]
---

# Reach for a Macro Only When a Function Cannot Express It [HIGH]

## Description
A macro operates purely on tokens, before the compiler has any notion of types, which means it sits outside everything type inference and IDE tooling normally provide — jump-to-definition, accurate autocomplete, an error that names the actual problem instead of the expanded code where it surfaced. It's also a real (if usually small) tax on incremental build times, and unlike a function, a macro can't be stored in a variable, passed as an argument, or referred to as a value at all. None of that is a reason to avoid macros entirely, but it does mean a generic function should be the default whenever it can do the same job — reserving macros for the handful of things a function genuinely cannot do: taking a variable number of arguments, implementing an embedded DSL, generating a trait implementation across a set of types that isn't known ahead of time, or checking a format string's structure at compile time.

## Bad Example
```rust
// Nothing here requires a macro: fixed arity, no DSL, no trait impl needed
macro_rules! double {
    ($x:expr) => {
        $x * 2
    };
}

fn main() {
    let n = double!(21);
    println!("{n}");
}
```

## Good Example
```rust
// A generic function is clearer, debuggable with a real stack frame,
// and just as efficient once inlined
#[inline]
fn double<T>(x: T) -> T
where
    T: std::ops::Add<Output = T> + Copy,
{
    x + x
}

fn main() {
    let n = double(21_i32);
    println!("{n}");
}
```

## Notes
- Good candidates for a macro include a genuinely variable-length argument list like `vec![]` or `println!`, implementing the same trait for a set of unrelated types that share no common bound, an embedded syntax that isn't valid Rust on its own (a SQL or regex literal, say), and boilerplate that a `#[derive(...)]` proc-macro could generate on a caller's behalf.
- A fixed number of arguments and no special syntax needs is a strong signal to write a function or a trait method instead — the compiler then type-checks that body exactly once, rather than re-checking a fresh expansion at every call site the macro produces.
- Every macro a codebase adds is one more thing a new contributor has to learn to mentally expand before they can follow what the code is actually doing when something breaks — that cost is real and worth weighing against whatever the macro saves in repetition.

## References
- [macro-fragment-specifiers](macro-fragment-specifiers.md)
- [type-generic-bounds](type-generic-bounds.md)
