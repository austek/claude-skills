---
title: Recognize Cloning Where a Borrow Would Have Worked
impact: MEDIUM
impactDescription: A frequent, cheap-looking mistake that compounds across a codebase
tags: [anti-pattern, ownership, cloning, code-smell]
---

# Recognize Cloning Where a Borrow Would Have Worked [MEDIUM]

## Description
`.clone()` compiles almost anywhere, which is exactly what makes it a trap: reaching for it whenever the borrow checker complains is a way to make code compile without actually understanding whether the ownership transfer it performs is needed. The telltale shape of this anti-pattern is a `.clone()` immediately followed by only read access — a comparison, a print, a pass into a function that never mutates or stores the value — none of which needed an owned copy in the first place. Each individual clone looks harmless in a diff; the pattern becomes a real cost when it's repeated across a codebase, showing up as steady allocator churn a profiler eventually flags without an obvious single culprit.

## Bad Example
```rust
fn summarize(report: Report) {
    // .clone() called reflexively to dodge a borrow-checker complaint,
    // then only ever read from — no owned copy was actually needed.
    log_report(report.clone());
    if report.clone().is_valid() {
        archive(report);
    }
}
```

## Good Example
```rust
fn summarize(report: &Report) {
    log_report(report);       // read-only, borrow is enough
    if report.is_valid() {    // read-only, no clone needed
        archive(report.clone()); // clone only where ownership is genuinely required
    }
}
```

## Notes
- The reflex to watch for: hitting a borrow-checker error and reaching for `.clone()` as the first fix rather than the last resort — check first whether a `&`/`&mut` reference, restructured control flow, or `Cow` actually solves it without copying.
- A repeated clone of the same value across a hot loop is a stronger version of the same mistake — moving the clone (or removing it) outside the loop turns O(n) allocations into O(1).
- `clippy::redundant_clone` catches a meaningful subset of this pattern automatically — enabling it in CI catches new instances before they land, though it won't catch every case a human reviewer would.

## References
- [own-borrow-over-clone](own-borrow-over-clone.md)
- [own-cow-conditional](own-cow-conditional.md)
- [own-arc-shared](own-arc-shared.md)
