---
title: Enable clippy::perf to Catch Avoidable Inefficiencies
impact: MEDIUM
impactDescription: Flags allocation and copy patterns that cost real cycles for no correctness benefit
tags: [tooling, clippy, performance, allocations]
---

# Enable clippy::perf to Catch Avoidable Inefficiencies [MEDIUM]

## Description
`clippy::perf` looks for code that spends more CPU or memory than the equivalent, equally-readable alternative would — an unnecessary `.to_string()` before a call that already accepts `impl Into<String>`, a `.contains("x")` string-pattern search where a `char` pattern is faster, a collect-then-iterate that could skip the intermediate `Vec`. None of these are algorithmic problems worth reaching for a profiler over; they're small, mechanical wastes that a reviewer might not notice by eye but that clippy catches instantly and for free on every build. Enabling the group is close to a strict improvement: the suggested fix is nearly always both faster and no less readable than the original.

## Bad Example
```rust
// Extra allocation just to satisfy Into<String> — take_string already
// accepts &str directly.
fn take_string(s: impl Into<String>) {}
take_string("hello".to_string());
```

## Good Example
```rust
take_string("hello");
```

## Notes
- `single_char_pattern` (use `'x'` instead of `"x"` for a one-character search) is one of the most common hits, and its fix is purely mechanical — the `char` form avoids the string-search machinery entirely for a single-character match.
- `unnecessary_to_owned` and `redundant_allocation` both point at the same underlying issue from different angles: a value gets copied or boxed a second time when the existing reference or owned value would already satisfy the call site.
- `Vec::with_capacity(n)` followed by `n` pushes avoids the repeated reallocation-and-copy that plain `Vec::new()` plus pushes incurs as the vector grows — `slow_vector_initialization` catches the pattern of filling a vector without ever giving it an initial capacity hint.
- These lints report real but usually small per-call costs — worth fixing because the fix is nearly free, but not a substitute for profiling before chasing a genuine performance problem in a hot path.

## References
- [lint-warn-complexity](lint-warn-complexity.md)
- [mem-with-capacity](../../rust-coding-standards/rules/mem-with-capacity.md)
- [perf-profile-first](../../rust-coding-standards/rules/perf-profile-first.md)
