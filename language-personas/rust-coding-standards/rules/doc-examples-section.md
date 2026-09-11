---
title: Give Every Non-Trivial API an # Examples Block
impact: MEDIUM
impactDescription: Turns docs into compiler-checked usage guarantees via doctests
tags: [documentation, examples, doctests, rustdoc]
---

# Give Every Non-Trivial API an # Examples Block [MEDIUM]

## Description
A description of what a function does rarely answers the question a new caller actually has, which is closer to "what does calling this look like." An `# Examples` section with a fenced Rust block answers that directly, and because rustdoc compiles and runs every such block as a doctest by default, the example can't silently rot the way a prose description can — rename the function, and the doctest breaks the build instead of quietly going stale. A struct or function with real usage nuance (multiple valid call shapes, an edge case worth calling out) benefits from more than one example block, each showing a different scenario.

## Bad Example
```rust
/// Splits a string on the first occurrence of a delimiter.
pub fn split_once(s: &str, delim: char) -> Option<(&str, &str)> {
    // no example: caller has to guess return shape, and what happens
    // when the delimiter isn't found
}
```

## Good Example
```rust
/// Splits a string on the first occurrence of a delimiter.
///
/// # Examples
///
/// ```
/// use my_crate::split_once;
///
/// assert_eq!(split_once("key=value", '='), Some(("key", "value")));
/// ```
///
/// Returns `None` when the delimiter isn't present:
///
/// ```
/// use my_crate::split_once;
///
/// assert_eq!(split_once("no-delimiter", '='), None);
/// ```
pub fn split_once(s: &str, delim: char) -> Option<(&str, &str)> {
    // ...
}
```

## Notes
- `cargo test --doc` runs every doctest in the crate — a broken example fails CI the same way a unit test would, which is exactly what makes examples trustworthy documentation instead of aspirational prose.
- Prefer `?` over `.unwrap()` inside example bodies for anything genuinely fallible (see `doc-question-mark`), and hide setup lines the reader doesn't need with the `# ` prefix (see `doc-hidden-setup`).
- A second example demonstrating the failure path (an `Err` or `None` case) is often more informative than a second example of the happy path repeated with different inputs.

## References
- [doc-question-mark](doc-question-mark.md)
- [doc-hidden-setup](doc-hidden-setup.md)
- [doc-errors-section](doc-errors-section.md)
