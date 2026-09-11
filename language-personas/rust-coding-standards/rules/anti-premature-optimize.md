---
title: Don't Add Optimization Complexity Before a Profiler Asks for It
impact: MEDIUM
impactDescription: Trades real, measured simplicity for a speedup nobody has confirmed exists
tags: [anti-pattern, performance, methodology, complexity]
---

# Don't Add Optimization Complexity Before a Profiler Asks for It [MEDIUM]

## Description
`unsafe` blocks to skip bounds checks, a hand-rolled cache, a custom data structure "for speed" — each of these is a real cost paid immediately (more code to maintain, more surface for bugs, more for the next reader to understand) in exchange for a performance benefit that, without a profiler confirming it, is only a guess. Most code isn't on a hot path at all; the parts that are usually aren't the parts intuition points to. Writing the simple, idiomatic version first and reaching for the standard library's already-optimized primitives costs nothing extra in either performance or clarity until a profiler shows a specific function actually needs the complexity.

## Bad Example
```rust
fn sum(data: &[i32]) -> i32 {
    // unsafe "for performance", with no measurement behind the claim
    unsafe {
        let mut total = 0;
        for i in 0..data.len() {
            total += *data.get_unchecked(i);
        }
        total
    }
}
```

## Good Example
```rust
fn sum(data: &[i32]) -> i32 {
    data.iter().sum() // simple, safe, and the compiler already optimizes this well
}

// If a flamegraph later shows this function as a genuine bottleneck,
// that's the point to revisit it with a benchmark proving the change helps —
// not before.
```

## Notes
- The standard library's collections and iterator methods are already heavily optimized — a hand-rolled `Vec` or a manual SIMD loop is competing against work the standard library has already done, usually without matching it.
- When a genuine hot path is confirmed by a profiler, document the optimization with the numbers that justified it (which benchmark, what improvement) directly in the code — a future reader shouldn't have to take the complexity on faith either.
- `#[inline(always)]` scattered speculatively across a codebase is one of the most common forms of this anti-pattern — the compiler's own inlining heuristics are usually better-informed than a guess made while writing the function.

## References
- [perf-profile-first](perf-profile-first.md)
- [test-criterion-bench](../../rust-testing/rules/test-criterion-bench.md)
- [opt-inline-small](opt-inline-small.md)
