---
title: Recognize the Collect-Then-Iterate-Again Anti-Pattern
impact: MEDIUM
impactDescription: A copy-paste-friendly mistake that multiplies allocations with each added step
tags: [anti-pattern, iterators, allocation, code-smell]
---

# Recognize the Collect-Then-Iterate-Again Anti-Pattern [MEDIUM]

## Description
This anti-pattern grows by accretion: a pipeline starts as one `.filter().collect()`, a second transformation gets added later as its own `.map().collect()` on the first result, and by the third addition nobody notices the function now allocates three separate collections to do what one lazy chain would do in a single pass. It's easy to introduce incrementally because each individual addition looks locally reasonable — "collect the filtered results, then map over them" reads fine in isolation — and the fix (deleting the intermediate `let` bindings and chaining the adapters directly) is usually a pure win with no downside once spotted.

## Bad Example
```rust
fn sum_valid(items: &[Item]) -> i64 {
    // Added incrementally: filter first, then a second pass to sum.
    let valid: Vec<_> = items.iter().filter(|i| i.is_valid()).collect();
    valid.iter().map(|i| i.value).sum() // second allocation, second pass
}
```

## Good Example
```rust
fn sum_valid(items: &[Item]) -> i64 {
    items.iter().filter(|i| i.is_valid()).map(|i| i.value).sum() // one pass, zero allocations
}
```

## Notes
- The giveaway in a diff or review is a `let name: Vec<_> = iter_chain.collect();` immediately followed by `name.iter()` or `name.into_iter()` starting a new chain — that's almost always mergeable into the original chain.
- A collection genuinely needed twice (sorted once, then iterated for output; or accessed by both `.len()` and by index) is not this anti-pattern — the marker is an intermediate `Vec` that exists only to feed the very next adapter.
- Returning `impl Iterator<Item = T>` from a helper instead of a concrete `Vec` sidesteps the temptation entirely — there's no intermediate collection to accidentally introduce when the function signature never produces one.

## References
- [perf-collect-once](perf-collect-once.md)
- [perf-iter-lazy](perf-iter-lazy.md)
- [perf-iter-over-index](perf-iter-over-index.md)
