---
title: Cross-Link Related Types to Build a Navigable Doc Graph
impact: LOW
impactDescription: Readers discover the rest of an API surface without leaving the current page
tags: [documentation, rustdoc, intra-doc-links, discoverability]
---

# Cross-Link Related Types to Build a Navigable Doc Graph [LOW]

## Description
A single item's documentation rarely tells the whole story — a `parse` function has a companion error enum, a `Config` struct has a `ConfigBuilder` that constructs it, an `Iterator` impl relates back to the trait's other methods. Intra-doc links between these related items turn a set of isolated pages into something a reader can actually explore: land on `Config`, follow a link to `ConfigBuilder`, follow another to `ConfigError`, without ever leaving rustdoc to search the crate. A dedicated `# Related` or `# See Also` section that lists these connections explicitly, rather than relying on the reader to notice them scattered through prose, makes that exploration close to free.

## Bad Example
```rust
/// A configuration builder.
pub struct Config { /* ... */ }

impl Config {
    /// Creates a new builder.
    pub fn builder() -> ConfigBuilder { /* ... */ }
}
```

## Good Example
```rust
/// Immutable, validated application configuration.
///
/// # Related
///
/// - [`ConfigBuilder`] — constructs a `Config` with validation
/// - [`ConfigError`] — the errors `ConfigBuilder::build` can return
pub struct Config { /* ... */ }

impl Config {
    /// Creates a new [`ConfigBuilder`] for assembling a `Config`.
    pub fn builder() -> ConfigBuilder { /* ... */ }
}
```

## Notes
- A module-level `//!` comment is a good place to list a module's main types and functions as links (`- [\`Parser\`] — the entry point`), giving `cargo doc` a real table of contents instead of an alphabetical item dump.
- Link to the specific trait method being implemented (`` [`Iterator::next`] ``) rather than just the trait, when an impl block's doc comment discusses one method in particular.
- `RUSTDOCFLAGS="-D warnings" cargo doc` catches a broken cross-link the same way it catches any other unresolved intra-doc link — the check doesn't care whether the link exists for navigation or as part of an API description.

## References
- [doc-intra-links](doc-intra-links.md)
- [doc-module-inner](doc-module-inner.md)
