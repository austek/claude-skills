---
title: Format the # Errors Section Consistently for Fallible Functions
impact: MEDIUM
impactDescription: Lets callers plan for failure without reading the implementation
tags: [documentation, error-handling, rustdoc, api-design]
---

# Format the # Errors Section Consistently for Fallible Functions [MEDIUM]

## Description
A `Result`-returning function's signature says failure is possible but never says *why* — a `# Errors` heading in the doc comment is where that missing half goes, and a consistent shape across a crate makes it fast to scan. Three shapes cover almost every case: a bullet list of independent conditions, one `Returns [\`Enum::Variant\`] if ...` line per variant when the error type is an enum with distinct causes, or a short paragraph when there's really only one condition worth naming. Whichever shape fits, `clippy::missing_errors_doc` can enforce that the section exists at all on any `pub fn` returning `Result`.

## Bad Example
```rust
/// Sends the request and parses the response.
pub fn send(req: Request) -> Result<Response, HttpError> {
    // caller has to read the body to learn there are four ways this fails
}
```

## Good Example
```rust
/// Sends the request and parses the response.
///
/// # Errors
///
/// - [`HttpError::Timeout`] if the server doesn't respond within the configured timeout
/// - [`HttpError::InvalidUrl`] if `req`'s URL fails to parse
/// - [`HttpError::ConnectionRefused`] if the server refuses the connection
/// - [`HttpError::Decode`] if the response body isn't valid for the expected type
pub fn send(req: Request) -> Result<Response, HttpError> {
    // ...
}
```

## Notes
- Pick the variant-mapped style whenever the error type is a real enum — it doubles as a table of contents a caller can `match` against directly.
- Link each named condition with an intra-doc link (`` [`HttpError::Timeout`] ``) so the reader can jump straight to that variant's own documentation instead of hunting for it.
- A function that both returns `Err` and can panic needs a `# Panics` section alongside `# Errors` — they're two different failure classes and conflating them hides one from the reader.

## References
- [doc-panics-section](doc-panics-section.md)
- [err-doc-errors](err-doc-errors.md)
- [doc-intra-links](doc-intra-links.md)
