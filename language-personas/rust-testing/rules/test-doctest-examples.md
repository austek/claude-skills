---
title: Keep Doc Examples as Executable Doctests
impact: MEDIUM
impactDescription: Catches documentation drift at compile time instead of leaving stale examples for users
tags: [testing, documentation, doctests, rustdoc]
---

# Keep Doc Examples as Executable Doctests [MEDIUM]

## Description
A code sample sitting inside a plain comment or an unfenced doc line is inert text — nothing checks whether it still compiles, let alone whether it still does what the prose claims. A ` ``` ` fenced block inside a `///` doc comment is different: `cargo test` compiles and runs it as its own tiny test, so the moment a function's signature or behavior changes, every doctest demonstrating the old shape fails the build. That single property — examples can't rot silently — is what makes doctests worth the extra ceremony over an ordinary comment. They also double as the example a user sees rendered on the crate's `rustdoc` page, so the same block is simultaneously user-facing documentation and a regression test.

## Bad Example
```rust
/// Parses a number from a string.
///
/// Example:
/// let n = parse("42");  // written as a comment, never compiled or run
/// assert_eq!(n, 42);
pub fn parse(s: &str) -> i32 {
    s.parse().unwrap()
}
```

## Good Example
```rust
/// Parses a number from a string.
///
/// # Examples
///
/// ```
/// use my_crate::parse;
///
/// let n = parse("42");
/// assert_eq!(n, 42);
/// ```
pub fn parse(s: &str) -> i32 {
    s.parse().unwrap()
}
```

## Notes
- Lines prefixed with `#` inside the fenced block are compiled and run but hidden from the rendered documentation — use this to hide temp-file or import boilerplate that would distract from the point of the example, as in `# use std::io::Write;`.
- An example that needs to demonstrate error handling can end with `# Ok::<(), my_crate::Error>(())` so the whole block can use `?` while still type-checking as a doctest.
- Use `no_run` for examples that compile but would hang or have side effects if actually executed (starting a server), `ignore` for platform-specific snippets that can't compile everywhere, and `compile_fail` to assert that a snippet is *supposed* to fail to compile — each still gets checked to the degree its annotation allows.
- Run only the doctests with `cargo test --doc` when iterating on documentation without re-running the full unit-test suite.

## References
- [doc-examples-section](../../rust-coding-standards/rules/doc-examples-section.md)
- [doc-hidden-setup](../../rust-coding-standards/rules/doc-hidden-setup.md)
- [doc-question-mark](../../rust-coding-standards/rules/doc-question-mark.md)
