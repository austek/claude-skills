---
title: Enable clippy::style for Idiomatic Patterns
impact: LOW
impactDescription: Nudges non-idiomatic-but-correct code toward the pattern most Rust readers expect
tags: [tooling, clippy, style, idioms]
---

# Enable clippy::style for Idiomatic Patterns [LOW]

## Description
`clippy::style` doesn't catch bugs or inefficiencies — it catches code that works fine but doesn't match the idiom most Rust developers would reach for automatically, like checking `.len() == 0` instead of `.is_empty()`, or writing a `match` with two arms that just re-wrap the same `Result` instead of returning it directly. The value isn't in any single fix; it's in reducing the number of small "why did they write it this way" moments a reviewer or new contributor hits while reading the codebase, since idiomatic code is faster to read precisely because it's what everyone already expects.

## Bad Example
```rust
// Redundant pattern match — this is just `result` wrapped and unwrapped
// for no reason.
match result {
    Ok(x) => Ok(x),
    Err(e) => Err(e),
}
```

## Good Example
```rust
result
```

## Notes
- `len_zero` (prefer `.is_empty()` over `.len() == 0`) exists because `is_empty()` is both clearer intent and sometimes cheaper — for some collection types, checking emptiness doesn't require the same work as computing a full length.
- `collapsible_if` and `single_match` both point at the same underlying habit: control flow written more verbosely than the condition actually requires, whether that's nested `if`s that could combine with `&&` or a two-arm `match` that's really an `if let`.
- Some style lints are genuinely a matter of house style rather than universal idiom — `needless_return`, for instance, some teams deliberately keep explicit `return` statements for clarity at function boundaries. `#[allow(clippy::needless_return)]` on those specific functions is reasonable; disabling the lint project-wide throws away the rest of the group's value for one preference.
- This group pairs naturally with `cargo fmt --check` in CI — formatting handles whitespace and layout, `clippy::style` handles the choice of construct, and together they cover most of what a manual style-focused review pass would otherwise need to catch by eye.

## References
- [lint-warn-suspicious](lint-warn-suspicious.md)
- [lint-warn-complexity](lint-warn-complexity.md)
- [lint-rustfmt-check](lint-rustfmt-check.md)
