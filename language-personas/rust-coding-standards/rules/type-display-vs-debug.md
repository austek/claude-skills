---
title: Use Display for Users and Debug for Diagnostics — Never Swap Them
impact: MEDIUM
impactDescription: Keeps user-facing messages clean and log output structurally useful
tags: [type-safety, display, debug, formatting]
---

# Use Display for Users and Debug for Diagnostics — Never Swap Them [MEDIUM]

## Description
`Debug` (`{:?}`) is for developers — logs, panic messages, test assertions, `dbg!()` — and should almost always be derived, reflecting the type's internal structure verbatim. `Display` (`{}`) is for end users — CLI output, human-readable error messages, log fields meant to be read by a person — and must be hand-written to describe the condition clearly. `std::error::Error` requires `Display`, so its output is what propagates through `anyhow::Context` and similar error-chaining tools. Routing `Debug` output to users leaks field names and internal representation; routing `Display` output into a log framework loses the structure a derived `Debug` would have preserved.

## Bad Example
```rust
#[derive(Debug)]
struct ParseError {
    input: String,
    line: u32,
}

// Mistake: user-facing message built from Debug output
fn report_error(e: &ParseError) {
    eprintln!("failed: {:?}", e); // leaks internal field names
}
```

## Good Example
```rust
use std::fmt;

#[derive(Debug)] // free diagnostic output
struct ParseError {
    input: String,
    line: u32,
}

impl fmt::Display for ParseError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "parse error on line {}: {}", self.line, self.input)
    }
}

impl std::error::Error for ParseError {}
```

## Notes
- Never derive `Display` — it must always be intentionally written to describe the condition for a human.
- `#[derive(Debug)]` on every public type is API Guidelines rule C-DEBUG; there's rarely a reason to skip it.
- If an error type implements `std::error::Error`, its `Display` becomes the human-readable message threaded through error-chaining crates — get it right once, not per call site.
- `{:#?}` pretty-printing is still `Debug`; reach for it in test assertion output, not in anything a user sees.

## References
- [api-common-traits](api-common-traits.md)
- [err-thiserror-lib](err-thiserror-lib.md)
- [type-numeric-fmt](type-numeric-fmt.md)
