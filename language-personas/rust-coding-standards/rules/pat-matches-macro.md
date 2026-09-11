---
title: Use matches! for Boolean Pattern Tests
impact: LOW
impactDescription: Replaces a verbose match-to-bool with a single clear expression
tags: [pattern-matching, readability, macros]
---

# Use matches! for Boolean Pattern Tests [LOW]

## Description
Asking "does this value match this shape?" as a yes-or-no question doesn't need a full `match` expression with a `true` arm and a `false` arm — that structure exists to compute different things for different variants, and using it just to produce a boolean buries a one-line question under several lines of boilerplate. `matches!(value, Pattern)` says the same thing directly: it evaluates to `true` or `false` depending on whether `value` fits `Pattern`, with nothing else to read. The verbose form is common enough that Clippy has a dedicated lint for it (`clippy::match_like_matches_macro`), and the macro itself is more capable than it might look, since it accepts an optional guard clause, which means a range check or a filtered `Option` test both fit in a single `matches!` call with no temporary binding needed to make the guard's variable visible.

## Bad Example
```rust
enum Status {
    Active,
    Pending,
    Closed,
}

fn is_active(s: &Status) -> bool {
    match s {
        Status::Active => true,
        _ => false,
    }
}

fn is_positive(opt: Option<i32>) -> bool {
    match opt {
        Some(v) if v > 0 => true,
        _ => false,
    }
}
```

## Good Example
```rust
enum Status {
    Active,
    Pending,
    Closed,
}

fn is_active(s: &Status) -> bool {
    matches!(s, Status::Active)
}

fn is_positive(opt: Option<i32>) -> bool {
    matches!(opt, Some(v) if v > 0)
}

// Combining multiple variants with `|`
fn is_terminal(s: &Status) -> bool {
    matches!(s, Status::Closed | Status::Pending)
}

// Pairs naturally with is_/has_/can_ predicate methods
impl Status {
    pub fn is_terminal(&self) -> bool {
        matches!(self, Self::Closed | Self::Pending)
    }
}
```

## Notes
- The `clippy::match_like_matches_macro` lint ships as part of `clippy::style`, so a project running Clippy's default lint groups is likely already catching new instances of the verbose form without any extra configuration.
- Rust 1.96 stabilized `assert_matches!` in `std::assert_matches` (it's not in the prelude, so it needs an explicit `use`), and it's worth reaching for in tests specifically because a failed assertion prints the actual value via `Debug` — a much more useful failure message than the bare boolean `assert!(matches!(...))` produces.
- `matches!` only ever answers a yes/no question about shape — the moment a branch needs to extract and use the value the pattern bound, or needs to do something different per-variant rather than a single true/false, a real `match` or `if let` is the correct tool instead.

## References
- [pat-exhaustive-enum](pat-exhaustive-enum.md)
