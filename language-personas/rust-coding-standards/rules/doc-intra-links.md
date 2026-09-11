---
title: Link to Items with Intra-Doc Link Syntax, Not Plain Text
impact: LOW
impactDescription: Makes references clickable and rustdoc-verified instead of silently stale
tags: [documentation, rustdoc, intra-doc-links, navigation]
---

# Link to Items with Intra-Doc Link Syntax, Not Plain Text [LOW]

## Description
Writing `` `Buffer` `` or plain `Buffer` in a doc comment renders as inert text — a reader has to already know where `Buffer` lives to find it. Writing `` [`Buffer`] `` instead produces a clickable link in the generated HTML, and rustdoc checks at build time that the target actually exists, so a rename that breaks the reference surfaces as a build warning rather than a dead link nobody notices for months. The syntax covers more than bare type names: `[Self::method]` reaches a method on the current type, `[Type::CONST]` reaches an associated constant, and a `fn@`/`struct@`/`trait@` prefix disambiguates when an item name is shared by more than one kind.

## Bad Example
```rust
/// Returns the number of bytes currently stored.
///
/// See also capacity() for the allocated size, and the Buffer
/// struct for the full API.
pub fn len(&self) -> usize {
    self.data.len()
}
```

## Good Example
```rust
/// Returns the number of bytes currently stored.
///
/// See also [`capacity`](Self::capacity) for the allocated size, and
/// [`Buffer`] for the full API.
pub fn len(&self) -> usize {
    self.data.len()
}
```

## Notes
- `RUSTDOCFLAGS="-D warnings" cargo doc --no-deps` (or `[lints.rustdoc] broken_intra_doc_links = "deny"` in `Cargo.toml`) turns an unresolved link into a hard failure in CI instead of a warning someone has to notice.
- Disambiguation prefixes (`fn@foo`, `mod@foo`, `struct@Error`, `trait@Error`) matter whenever a function, module, or type shares a name with something else in scope — without one, rustdoc has to guess which item the link means.
- A reference-style link (`` [`Result`]: std::result::Result `` on its own line) keeps a long or repeated path out of the middle of a sentence, which is useful once the same item gets linked several times in one doc comment.

## References
- [doc-all-public](doc-all-public.md)
- [doc-errors-section](doc-errors-section.md)
