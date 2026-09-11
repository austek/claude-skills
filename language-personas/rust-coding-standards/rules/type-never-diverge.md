---
title: Use the Never Type for Functions That Never Return
impact: LOW
impactDescription: Documents non-returning control flow and coerces cleanly into any match arm
tags: [type-safety, never-type, control-flow, diverging]
---

# Use the Never Type for Functions That Never Return [LOW]

## Description
`!`, the never type, marks a function that never returns normally — it loops forever, panics, or exits the process. Declaring `-> !` documents that guarantee in the signature instead of leaving callers to infer it, and it lets the compiler coerce a diverging expression to whatever type the surrounding context needs, which is exactly why `panic!()` and `std::process::exit()` slot cleanly into a `match` arm expecting a concrete value. Using `-> ()` (or an unrelated `Option`/`Result`) for a function that in practice never returns hides that fact from both readers and the compiler.

## Bad Example
```rust
// Implicit () return type, even though this never actually returns.
fn infinite_loop() {
    loop {
        process_events();
    }
}
```

## Good Example
```rust
// ! documents the guarantee and lets panic!()/process::exit() coerce cleanly.
fn infinite_loop() -> ! {
    loop {
        process_events();
    }
}

fn get_value(opt: Option<i32>) -> i32 {
    match opt {
        Some(v) => v,
        None => panic!("no value"), // panic! is `!`, coerces to i32
    }
}
```

## Notes
- `fn f() -> !` as a return type has been stable since Rust 1.41 — no feature gate needed.
- Using `!` as an arbitrary type argument (e.g. `Result<(), !>`) still requires the nightly `never_type` feature; on stable, use `std::convert::Infallible` as the conventional stand-in.
- Standard library examples: `std::process::exit`, the `panic!` macro, `unsafe fn unreachable_unchecked`, and `unreachable!()`.
- Reach for `-> !` on any function whose only paths are `loop {}` with no `break`, an unconditional `panic!`, or an unconditional process exit.

## References
- [err-result-over-panic](err-result-over-panic.md)
- [type-result-fallible](type-result-fallible.md)
- [opt-cold-unlikely](opt-cold-unlikely.md)
