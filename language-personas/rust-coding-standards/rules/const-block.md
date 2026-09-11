---
title: Use const Blocks for Compile-Time Evaluation and Assertions
impact: MEDIUM
impactDescription: Moves invariant checks from runtime panics to build failures, zero runtime cost
tags: [const, compile-time, assertions, zero-cost]
---

# Use const Blocks for Compile-Time Evaluation and Assertions [MEDIUM]

## Description
Since Rust 1.79, writing `const { ... }` anywhere an expression is expected tells the compiler to fully evaluate that expression during compilation, rather than waiting for the surrounding function to actually run. The clearest payoff is turning an `assert!` that only depends on compile-time-known values into a build failure instead of a runtime panic — a broken invariant is caught before the binary ships, with a diagnostic pointing straight at the line. The same syntax also works as a lightweight way to precompute a one-off value exactly where it's used, without declaring a separate top-level `const` item, and it's the mechanism that lets each element of an array of non-`Copy` values carry its own constant expression during initialization. None of this costs anything once the program is actually running, because the block itself never executes at runtime — only its result does.

## Bad Example
```rust
const SIZE: usize = 64;

fn process(buf: &[u8]) {
    // Runtime panic — the invariant only surfaces when this code path executes
    assert!(SIZE.is_power_of_two(), "SIZE must be a power of two");
    assert!(buf.len() <= SIZE);
}

struct Packet<const HDR: usize, const BODY: usize>;

impl<const HDR: usize, const BODY: usize> Packet<HDR, BODY> {
    fn new() -> Self {
        // Also deferred to runtime, even though HDR and BODY are known at compile time
        assert!(HDR + BODY <= 1500, "packet exceeds ethernet MTU");
        Packet
    }
}
```

## Good Example
```rust
const SIZE: usize = 64;

fn process(buf: &[u8]) {
    // Build fails immediately if SIZE ever changes to a non-power-of-two
    const { assert!(SIZE.is_power_of_two(), "SIZE must be a power of two") };
    // buf.len() is only known at runtime, so this assertion stays dynamic
    assert!(buf.len() <= SIZE);
}

struct Packet<const HDR: usize, const BODY: usize>;

impl<const HDR: usize, const BODY: usize> Packet<HDR, BODY> {
    fn new() -> Self {
        // HDR and BODY are compile-time constants — check them at compile time
        const { assert!(HDR + BODY <= 1500, "packet exceeds ethernet MTU") };
        Packet
    }
}

// Inline const block used as a value — evaluated once, inlined at the call site
fn magic_header() -> u32 {
    const { 0xDEAD_BEEFu32.swap_bytes() }
}
```

## Notes
- Think of `const { ... }` as "evaluate this now" rather than as a declaration — it fits anywhere an ordinary expression fits: a function body, a match arm, an array initializer, a default value.
- It has no name and no lifetime beyond the expression that contains it, unlike a top-level `const NAME: T = expr;` item. Use the named form for a value shared across a crate; reach for the inline block when the guarantee only matters at one call site.
- Everything referenced inside the block must be knowable before compilation finishes — const generics, other `const` items, plain arithmetic on literals. A check that depends on a runtime argument such as `buf.len()` can never move into a `const` block and stays a regular `assert!`.

## References
- [const-fn](const-fn.md)
- [mem-assert-type-size](mem-assert-type-size.md)
