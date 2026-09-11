---
title: Enable clippy::suspicious to Catch Likely Bugs
impact: HIGH
impactDescription: Surfaces syntactically-valid code that almost never does what its author intended
tags: [tooling, clippy, suspicious, bugs]
---

# Enable clippy::suspicious to Catch Likely Bugs [HIGH]

## Description
`clippy::suspicious` sits between `correctness` (definitely wrong) and `style` (a matter of taste) — every lint in it flags a pattern that compiles fine and is occasionally intentional, but is wrong often enough that it deserves a second look every time. An operator-precedence trap like `1 << 4 + 1` (parsed as `1 << (4 + 1)`, probably not what was meant), a `map()` call used purely for its side effects instead of `for_each()`, or `--x` (which in Rust means double negation, not pre-decrement, because Rust has no `--` operator) are all things a reviewer skimming a diff is likely to read right past. The lint group exists precisely because these patterns look innocuous at a glance and only reveal themselves as bugs when someone traces through the actual operator precedence or semantics.

## Bad Example
```rust
// Reads like "shift left by 5", but + binds looser than <<:
// this is actually 1 << (4 + 1), i.e. 1 << 5.
let bits = 1 << 4 + 1;
```

## Good Example
```rust
// Parenthesize to make the intended grouping unambiguous, whichever it is.
let bits = (1 << 4) + 1;
```

## Notes
- `suspicious_map` catches a `.map(|x| { println!(...); x })` pattern where the closure exists for a side effect rather than a transformation — `for_each` is the correct tool when the point is running code per element, not producing a new iterator.
- `--x` is a classic trap for anyone coming from C-family languages: Rust has no decrement operator, so `--x` parses as unary negation applied twice, which is just `x`. The lint flags this because it's almost never what the author meant to write.
- `suspicious_arithmetic_impl` and `suspicious_op_assign_impl` fire on custom `Add`/`Mul`/etc. implementations that use an operator other than the one being implemented (a `Mul` impl that internally does addition, say) — legitimate in niche cases like reduction-based matrix multiplication, in which case a scoped `#[allow]` with a comment explaining why is the right response rather than disabling the lint.
- Because every lint in this group represents "probably a bug, occasionally not," resist reaching for a blanket `#[allow(clippy::suspicious)]` — silence individual false positives at the smallest scope so the rest of the group keeps doing its job.

## References
- [lint-deny-correctness](lint-deny-correctness.md)
- [lint-warn-style](lint-warn-style.md)
- [lint-warn-complexity](lint-warn-complexity.md)
