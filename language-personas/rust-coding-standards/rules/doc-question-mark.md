---
title: Propagate Errors with ? in Doc Examples, Not unwrap
impact: MEDIUM
impactDescription: Makes a doctest fail loudly instead of masking a broken example with unwrap
tags: [documentation, examples, doctests, error-handling]
---

# Propagate Errors with ? in Doc Examples, Not unwrap [MEDIUM]

## Description
An example is also a teaching tool, and `.unwrap()` teaches the habit of discarding errors rather than handling them — exactly the pattern most Rust style guides warn production code away from. `?` demonstrates the idiom callers should actually copy, and it has a practical side effect: rustdoc wraps a doctest using `?` in a function returning `Result`, so if the example's call genuinely fails, the doctest fails the build instead of panicking with a generic "called `unwrap()` on an `Err`" message that says nothing about which line broke.

## Bad Example
```rust
/// Loads configuration from a file.
///
/// # Examples
///
/// ```
/// let config = Config::from_file("app.toml").unwrap();
/// println!("{:?}", config.database_url);
/// ```
pub fn from_file(path: &str) -> Result<Config, Error> {
    // ...
}
```

## Good Example
```rust
/// Loads configuration from a file.
///
/// # Examples
///
/// ```
/// # use my_crate::{Config, Error};
/// # fn main() -> Result<(), Error> {
/// let config = Config::from_file("app.toml")?;
/// println!("{:?}", config.database_url);
/// # Ok(())
/// # }
/// ```
pub fn from_file(path: &str) -> Result<Config, Error> {
    // ...
}
```

## Notes
- The wrapping `fn main() -> Result<(), E>` can stay implicit — ending the block with `# Ok::<(), my_crate::Error>(())` works without a visible `fn main` at all, which keeps the rendered example shorter.
- `.unwrap()` is still reasonable for a value that provably cannot fail in the shown context — a `Regex::new` call on a string literal known to be valid, or `"42".parse::<i32>().unwrap()` — but that's a narrow exception, not the default.
- Combine this with `doc-hidden-setup` to keep the `# fn main`/`# Ok(())` scaffolding out of the rendered example while it still compiles as part of the doctest.

## References
- [doc-examples-section](doc-examples-section.md)
- [doc-hidden-setup](doc-hidden-setup.md)
- [err-question-mark](err-question-mark.md)
