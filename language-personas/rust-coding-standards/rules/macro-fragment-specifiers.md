---
title: Capture with Precise Fragment Specifiers, Not Raw tt
impact: MEDIUM
impactDescription: Produces targeted compiler errors instead of accepting malformed input silently
tags: [macros, declarative-macros, error-messages]
---

# Capture with Precise Fragment Specifiers, Not Raw tt [MEDIUM]

## Description
Every capture inside a `macro_rules!` pattern is written as `$name:specifier`, and the specifier is doing more work than it might look like — it tells the macro matcher exactly what grammatical category of Rust syntax is allowed there, which means the matcher itself can reject something that doesn't fit before the macro body ever runs. A capture written `:expr` will refuse anything that isn't a valid expression right at the call site, with an error that says so in plain terms. The generic `:tt` specifier, by contrast, matches almost any single token or bracketed group without checking what it actually represents, which pushes the real validation work into the macro body — and by the time something goes wrong there, the error tends to describe internal expansion details rather than the mistake the caller actually made.

## Bad Example
```rust
// Slurps everything as :tt, then re-expands it as if it were an expression
macro_rules! debug_val {
    ($($t:tt)*) => {
        println!("{} = {:?}", stringify!($($t)*), $($t)*);
    };
}

fn main() {
    debug_val!(1 + 2);      // works by accident
    debug_val!(let x = 1);  // accepted by the macro; fails deep inside expansion instead
}
```

## Good Example
```rust
macro_rules! debug_val {
    // :expr captures exactly one expression; the follow-set allows `=>` and `,` after it
    ($e:expr) => {
        println!("{} = {:?}", stringify!($e), $e);
    };
}

fn main() {
    debug_val!(1 + 2);
    // debug_val!(let x = 1); // now correctly rejected at the call site
}

// Optional trailing comma in a repetition, without a follow-set workaround
macro_rules! my_vec {
    ($($e:expr),* $(,)?) => {
        vec![$($e),*]
    };
}
let v = my_vec![1, 2, 3,];
```

## Notes
- The specifier vocabulary covers most of what a macro pattern needs: `:expr` for an expression, `:ty` for a type, `:ident` for a bare identifier, `:pat`/`:pat_param` for a pattern, plus `:path`, `:literal`, `:block`, `:stmt`, `:meta`, `:vis`, and `:lifetime` for their respective syntax categories. `:tt` is worth keeping in reserve for cases that genuinely need it, such as a recursive macro that munges tokens one at a time.
- A handful of specifiers — `:expr`, `:ty`, `:pat` among them — restrict what token is allowed to come immediately after the capture, typically to something like `=>`, `,`, `;`, or `|`. It's worth designing a macro's separators around that restriction up front, rather than writing the pattern first and discovering the restriction from a confusing compiler error.
- To accept a trailing comma in a repeated capture without running into that follow-set restriction, `$(,)?` placed right after the repetition is the standard idiom — it sits in separator position, so it doesn't collide with the restriction the way a comma written some other way might.

## References
- [macro-rules-hygiene](macro-rules-hygiene.md)
- [macro-prefer-functions](macro-prefer-functions.md)
