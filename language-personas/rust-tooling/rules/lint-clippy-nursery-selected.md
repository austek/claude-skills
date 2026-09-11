---
title: Cherry-Pick clippy::nursery Lints Instead of the Whole Group
impact: MEDIUM
impactDescription: Captures high-value nursery lints without the noise of unfinished ones
tags: [tooling, clippy, nursery, lints]
---

# Cherry-Pick clippy::nursery Lints Instead of the Whole Group [MEDIUM]

## Description
`clippy::nursery` holds lints that clippy's maintainers consider correct in principle but not yet mature enough for `clippy::all` — some still have rough edges, over-fire on valid code, or are actively being reworked between clippy releases. Turning on the entire group with `#![warn(clippy::nursery)]` means inheriting all of that instability at once: every clippy upgrade risks new false positives or lint behavior changes you didn't ask for. Enabling specific nursery lints individually — the ones that have proven reliable in practice, like `significant_drop_tightening` for lock guards held too long or `use_self` for repeated type names inside `impl` blocks — gets the value without signing up for the group's ongoing churn.

## Bad Example
```toml
# Cargo.toml — pulls in every nursery lint, proven and unproven alike.
[lints.clippy]
nursery = "warn"
```

## Good Example
```toml
# Cargo.toml — a deliberately curated starting set.
[lints.clippy]
significant_drop_tightening = "warn" # lock/guard held longer than needed
redundant_clone = "warn"             # .clone() where a move would do
use_self = "warn"                    # TypeName inside its own impl block should be Self
redundant_else = "warn"              # else after a diverging if branch
```

```rust
impl MyStruct {
    fn new() -> MyStruct {   // use_self fires: should be `-> Self`
        MyStruct { value: 0 }
    }
}
```

## Notes
- `significant_drop_tightening` is particularly worth enabling in async code, since a lock guard held across an `.await` point (rather than dropped before it) is a common source of accidental contention or deadlock.
- Treat the nursery group as a lint catalog to sample from, not a toggle to flip — review what a candidate lint flags across the actual codebase before adding it, since "correct but unpolished" sometimes means genuinely noisy in specific patterns your code happens to use.
- Because nursery lints move between releases — graduating to a stable group, getting reworked, or occasionally being removed — pin the clippy version used in CI so a nursery-lint change doesn't silently alter what counts as a passing build.
- `redundant_clone` catches clones that a move would satisfy just as well; it pairs naturally with ownership-focused review, since the fix is usually to stop borrowing and just move the value.

## References
- [lint-pedantic-selective](lint-pedantic-selective.md)
- [lint-warn-perf](lint-warn-perf.md)
- [anti-lock-across-await](../../rust-coding-standards/rules/anti-lock-across-await.md)
