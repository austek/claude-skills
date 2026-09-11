---
title: Don't Let panic! Stand In for an Ordinary Failure Path
impact: HIGH
impactDescription: Turns a routine, recoverable condition into a program crash
tags: [anti-pattern, error-handling, panics, code-smell]
---

# Don't Let panic! Stand In for an Ordinary Failure Path [HIGH]

## Description
A panic is meant for the situations a program genuinely cannot continue past — a broken invariant, corrupted internal state, a bug. Network requests timing out, files not existing, and users typing malformed input are none of those things; they're conditions any program touching the outside world will hit routinely, and handling them is part of the program's actual job, not an exceptional circumstance. Code that reaches for `panic!`, `.unwrap()`, or `.expect()` on these paths is treating "the request failed" the same as "something is fundamentally broken," and the caller pays for that conflation with a crash instead of a `Result` it could have acted on.

## Bad Example
```rust
fn process_age(age: i32) {
    if age < 0 {
        panic!("age cannot be negative"); // this is validation, not a bug
    }
}

fn find_or_die(items: &[Item], id: u64) -> &Item {
    items.iter().find(|i| i.id == id)
        .unwrap_or_else(|| panic!("item {id} not found")) // "not found" is an expected outcome
}
```

## Good Example
```rust
fn process_age(age: i32) -> Result<(), ValidationError> {
    if age < 0 {
        return Err(ValidationError::NegativeAge);
    }
    Ok(())
}

fn find(items: &[Item], id: u64) -> Option<&Item> {
    items.iter().find(|i| i.id == id) // absence is a normal, representable outcome
}
```

## Notes
- The test that separates a legitimate panic from this anti-pattern: is the condition something *the program's own logic* violated (an index computed wrong, a state machine reaching an impossible transition), or something the *outside world* produced (user input, network response, file contents)? The first is a bug and can panic; the second needs `Result` or `Option`.
- `assert!`/`unreachable!` guarding a genuine internal invariant, and a test asserting expected behavior with `panic!`, are the two contexts where reaching for a panic is still correct — neither represents an externally-triggerable condition.
- A caller that wants to convert an `Err` into a crash at a specific boundary (a `main` that exits on startup failure, say) can still do that explicitly with `.expect()` at that one boundary — the anti-pattern is baking the panic into the library-level function itself, removing the caller's ability to choose.

## References
- [err-result-over-panic](err-result-over-panic.md)
- [anti-unwrap-abuse](anti-unwrap-abuse.md)
- [err-expect-bugs-only](err-expect-bugs-only.md)
