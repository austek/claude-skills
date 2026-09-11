---
title: Avoid Iterator::chain Inside a Hot Inner Loop
impact: LOW
impactDescription: Removes a per-item branch that a hot loop pays on every iteration
tags: [performance, iterators, hot-path]
---

# Avoid Iterator::chain Inside a Hot Inner Loop [LOW]

## Description
`Iterator::chain` works by holding two iterators and, on every call to `.next()`, checking whether the first one is exhausted yet before falling through to the second. That check is one branch per element, which is invisible almost everywhere it's used — but inside a loop that runs millions of times per second, a branch the CPU can't always predict well adds up. The fix isn't to avoid `chain` categorically; it's to keep it out of the innermost, highest-iteration-count loop specifically, either by writing two separate loops over the two sources or by combining them into one contiguous collection before entering the hot section.

## Bad Example
```rust
// Chain re-checked on every one of a million iterations.
fn process_hot_path(a: &[i32], b: &[i32]) -> i64 {
    let mut sum = 0i64;
    for _ in 0..1_000_000 {
        for x in a.iter().chain(b.iter()) {
            sum += *x as i64;
        }
    }
    sum
}
```

## Good Example
```rust
// Two branch-free loops instead of one branchy one.
fn process_hot_path(a: &[i32], b: &[i32]) -> i64 {
    let mut sum = 0i64;
    for _ in 0..1_000_000 {
        for x in a { sum += *x as i64; }
        for x in b { sum += *x as i64; }
    }
    sum
}
```

## Notes
- `chain` is fine, even in performance-sensitive code, outside a repeatedly-executed inner loop — a one-time `a.into_iter().chain(b).collect()` or a `find()` that short-circuits on the first match never pays the per-item cost enough times to matter.
- When the two sources are combined more than once, pre-merging them outside the hot section (`Vec::with_capacity` + two `extend_from_slice` calls) removes the branch entirely rather than just moving it.
- Chaining three or more iterators compounds the branch count per element — a nested `chain(a).chain(b).chain(c)` inside a hot loop is a stronger version of the same problem, and a bigger win to fix.

## References
- [perf-iter-over-index](perf-iter-over-index.md)
- [perf-extend-batch](perf-extend-batch.md)
- [opt-cache-friendly](opt-cache-friendly.md)
