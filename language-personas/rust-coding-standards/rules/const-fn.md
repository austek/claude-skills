---
title: Mark Pure Functions const fn When They Can Run at Compile Time
impact: MEDIUM
impactDescription: Widens callable contexts to array lengths and const generics at zero runtime cost
tags: [const, compile-time, functions, zero-cost]
---

# Mark Pure Functions const fn When They Can Run at Compile Time [MEDIUM]

## Description
Adding the `const` qualifier to a function doesn't change what it does — it changes where the compiler is allowed to call it. A `const fn` remains an ordinary function when called with runtime values, but it also becomes legal to invoke from an array-length expression, a `const`/`static` initializer, or a const-generic argument, none of which can call a normal function. Whenever the call site actually needs that value at compile time, the compiler runs the function itself during compilation and bakes in the result, so the computation contributes nothing to the compiled binary. Most day-to-day function bodies already qualify: arithmetic, bitwise operators, `if`/`match`, loops, and slice indexing are all permitted in a `const fn` on stable Rust today. What still doesn't work is heap allocation and calling most trait methods, since those require runtime machinery the compiler can't fully collapse away.

## Bad Example
```rust
// Not const — the result cannot be used as an array length
fn header_len() -> usize {
    4
}

fn make_buf() -> [u8; 8] {
    // error: `header_len` is not a `const fn`
    [0u8; header_len()]
}

fn align_up(n: usize, align: usize) -> usize {
    (n + align - 1) & !(align - 1)
}

// Cannot be a const initializer, so this gets recomputed at every call site
fn aligned_size() -> usize {
    align_up(13, 8)
}
```

## Good Example
```rust
const fn header_len() -> usize {
    4
}

// Usable as an array length — evaluated at compile time
let buf = [0u8; header_len()];

const fn align_up(n: usize, align: usize) -> usize {
    (n + align - 1) & !(align - 1)
}

// Usable in a const initializer — computed once, at compile time
const ALIGNED: usize = align_up(13, 8); // 16

// Usable in a static array length too
static HEADER: [u8; header_len()] = [0u8; header_len()];
```

## Notes
- Marking a public function `const` is a non-breaking change for downstream users, so there's little cost to defaulting to it — write `const fn` for any pure, allocation-free function up front, and only remove the qualifier if a capability the body needs turns out to still be `const`-unstable.
- Nothing forces compile-time evaluation just because a function is `const` — an ordinary runtime call to a `const fn` behaves exactly like calling a regular function. The compiler only evaluates it eagerly when the surrounding context demands a constant, such as sizing an array or initializing a `static`.
- Check the feature against your crate's minimum supported Rust version before relying on it in a `const fn` body — not every `const`-eval capability that's stable on the latest compiler has been stable long enough to assume everywhere.

## References
- [const-block](const-block.md)
- [const-generics](const-generics.md)
- [opt-inline-small](opt-inline-small.md)
