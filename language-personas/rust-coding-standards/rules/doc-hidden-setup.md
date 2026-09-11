---
title: Hide Doctest Setup Lines Behind a # Prefix
impact: LOW
impactDescription: Keeps rendered examples focused on the API being taught
tags: [documentation, examples, doctests, rustdoc]
---

# Hide Doctest Setup Lines Behind a # Prefix [LOW]

## Description
A doctest usually needs a handful of lines — imports, constructing a fixture, unwrapping a `Result` at the end — that have nothing to do with the point the example is making but are still required for it to compile and run. Prefixing those lines with `# ` keeps them in the code rustdoc actually compiles and executes while dropping them from the rendered HTML, so a reader sees only the two or three lines that demonstrate the API, not the ten lines of scaffolding around them. The mechanism is purely cosmetic in rustdoc's output — nothing about test execution changes, only what gets displayed.

## Bad Example
```rust
/// # Examples
///
/// ```
/// use my_crate::{Client, ClientConfig};
/// use std::time::Duration;
///
/// let config = ClientConfig { timeout: Duration::from_secs(5), retries: 2 };
/// let client = Client::new(config);
///
/// // buried after five lines of setup:
/// let response = client.get("/status")?;
/// assert!(response.is_success());
/// # Ok::<(), my_crate::Error>(())
/// ```
```

## Good Example
```rust
/// # Examples
///
/// ```
/// # use my_crate::{Client, ClientConfig};
/// # use std::time::Duration;
/// # let config = ClientConfig { timeout: Duration::from_secs(5), retries: 2 };
/// # let client = Client::new(config);
/// let response = client.get("/status")?;
/// assert!(response.is_success());
/// # Ok::<(), my_crate::Error>(())
/// ```
```

## Notes
- Rendered, the reader sees only `let response = client.get("/status")?;` and the assertion — the construction and imports stay compiled but invisible.
- Don't hide setup that *is* the point of the example — a constructor or builder-pattern doc comment should show its own construction code in full rather than hiding it behind `#`.
- ` ```no_run ` and ` ```ignore ` solve a different problem than hidden setup: they stop a block from executing at all (useful for a long-running server example), whereas `# ` only controls what's rendered while the block still compiles and runs normally.

## References
- [doc-examples-section](doc-examples-section.md)
- [doc-question-mark](doc-question-mark.md)
