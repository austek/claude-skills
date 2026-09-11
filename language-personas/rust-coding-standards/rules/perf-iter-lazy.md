---
title: Keep an Iterator Pipeline Lazy Until the Final Consumer
impact: MEDIUM
impactDescription: Enables single-pass processing and lets a search short-circuit instead of scanning everything
tags: [performance, iterators, laziness]
---

# Keep an Iterator Pipeline Lazy Until the Final Consumer [MEDIUM]

## Description
Adapters like `.filter()`, `.map()`, and `.take()` don't do any work when called — they build a description of work to be done, and nothing actually runs element-by-element until a consuming method (`.collect()`, `.for_each()`, `.sum()`, `.any()`) drives the pipeline forward. That laziness is what makes chaining adapters cheap and what makes short-circuiting possible: `.find()` or `.any()` can stop at the first matching element instead of processing the whole input, but only if nothing upstream has already forced full evaluation into a concrete collection. Calling `.collect()` partway through a pipeline — to check `.is_empty()`, or just out of habit between steps — defeats both benefits at once.

## Bad Example
```rust
fn has_positive(data: &[i32]) -> bool {
    // Materializes every matching element just to ask if there's at least one.
    let positives: Vec<_> = data.iter().filter(|&&x| x > 0).collect();
    !positives.is_empty()
}
```

## Good Example
```rust
fn has_positive(data: &[i32]) -> bool {
    data.iter().any(|&x| x > 0) // stops at the first positive value
}

fn top_ten_doubled_positives(data: Vec<i32>) -> Vec<i32> {
    data.into_iter()
        .filter(|x| *x > 0)
        .map(|x| x * 2)
        .take(10) // pipeline can stop after 10 matches, no matter the input size
        .collect()
}
```

## Notes
- `.filter()`, `.map()`, `.zip()`, `.chain()`, `.flat_map()`, `.take()`, `.skip()`, and `.enumerate()` all return iterators without doing any element-level work — they're safe to chain freely.
- `.count()`, `.sum()`, `.fold()`, `.any()`, `.all()`, and `.find()` all consume the pipeline without ever materializing a collection — reach for these instead of `.collect()` followed by `.len()`/`.is_empty()`/manual iteration.
- Returning `impl Iterator<Item = T>` from a helper function preserves this laziness across a function boundary — the caller decides when (or whether) to actually drive the pipeline to completion, rather than the helper deciding for them by collecting internally.

## References
- [perf-collect-once](perf-collect-once.md)
- [perf-iter-over-index](perf-iter-over-index.md)
- [anti-collect-intermediate](anti-collect-intermediate.md)
