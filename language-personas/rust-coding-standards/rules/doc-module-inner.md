---
title: Give Every Module a //! Overview Comment
impact: LOW
impactDescription: Gives cargo doc module pages a real landing page instead of a bare item list
tags: [documentation, modules, rustdoc]
---

# Give Every Module a //! Overview Comment [LOW]

## Description
`//!` comments — inner doc comments — describe the module they appear inside, in contrast with `///` which describes whatever item comes right after it. Placed at the top of `lib.rs` or a module file, `//!` becomes the page a reader lands on in `cargo doc` before they've drilled into any individual struct or function, which makes it the right place to answer "what does this module provide and how do I get started" before the item-by-item reference takes over. An ordinary `//` comment above `mod auth;` documents nothing rustdoc will ever render — it's invisible to anyone reading generated docs rather than the source.

## Bad Example
```rust
// This module handles authentication: JWT and session-based.
mod auth;
```
```rust
// src/auth.rs — nothing here becomes part of the rendered docs
use std::collections::HashMap;

pub struct Session { /* ... */ }
```

## Good Example
```rust
//! Authentication and session utilities.
//!
//! Provides two strategies:
//!
//! - [`JwtAuth`] — stateless, token-based authentication
//! - [`SessionAuth`] — cookie-backed server-side sessions
//!
//! # Examples
//!
//! ```
//! use my_crate::auth::JwtAuth;
//!
//! let auth = JwtAuth::new("secret-key");
//! ```

use std::collections::HashMap;

pub struct Session { /* ... */ }
```

## Notes
- The crate root's `//!` block (top of `lib.rs`) becomes the front page of the entire crate's documentation — it's worth treating as the first thing a new user reads, not an afterthought.
- A useful module overview covers, roughly: a one-line summary, the module's main exported items (as intra-doc links), a quick-start example, and any feature flags that change what's available.
- `#![doc = include_str!("../README.md")]` at the crate root is an alternative to writing the crate-level `//!` block by hand — see `doc-crate-readme` for when that's the better fit.

## References
- [doc-all-public](doc-all-public.md)
- [doc-crate-readme](doc-crate-readme.md)
