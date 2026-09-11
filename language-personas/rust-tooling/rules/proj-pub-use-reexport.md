---
title: Use pub use to Flatten the Public API
impact: MEDIUM
impactDescription: Decouples the crate's internal directory layout from what consumers have to type
tags: [tooling, visibility, api-design, re-export]
---

# Use pub use to Flatten the Public API [MEDIUM]

## Description
A crate's internal module layout and its public API don't have to be the same shape — `pub use` re-exports an item at a shallower path than where it's actually defined, so `src/transport/http/client.rs::HttpClient` can be reachable to consumers as simply `my_crate::HttpClient`. Without re-exporting, users are forced to spell out the crate's internal organization in every import (`my_crate::transport::http::client::HttpClient`), which means any internal reshuffling — splitting a module, renaming a file, moving something to a different subdirectory — becomes a breaking change for every consumer, even though nothing about the type's actual behavior changed. Re-exporting the handful of items users actually need at the crate root keeps that internal freedom intact: you can reorganize `src/` however makes sense internally without ever touching the path a consumer imports from.

## Bad Example
```rust
// lib.rs — internal module paths leak directly into the public API.
pub mod transport;
pub mod auth;

// consumers are forced to know and type the internal structure:
use my_crate::transport::http::client::HttpClient;
use my_crate::auth::token::Token;
```

## Good Example
```rust
// lib.rs — modules stay private; only the re-exported names are public.
mod transport;
mod auth;

pub use transport::http::client::HttpClient;
pub use auth::token::Token;

// consumers write:
use my_crate::{HttpClient, Token};
```

## Notes
- Re-export selectively, not with a blanket `pub use internal::*;` for every module — naming exactly which items are re-exported is what makes the crate's real public surface legible from `lib.rs` alone, without cross-referencing every submodule.
- `pub use old_impl::Client as LegacyClient;` alongside `pub use new_impl::Client;` is a clean way to keep a deprecated implementation reachable under an explicit name during a migration, without the new and old types colliding under the same name.
- Re-exporting a type from an external dependency (`pub use bytes::Bytes;`) can be a deliberate API choice — it lets consumers use that type without adding the dependency themselves — but do it consciously, since it also means your crate's version now implicitly constrains which version of that dependency consumers can use.
- Glob re-exports (`pub use internal::*;`) are reasonable for consolidating an internal module's exports at the crate root, but avoid them for external crates specifically — `pub use serde::*;` pulls in far more than any consumer is likely to want and risks name collisions with your own public items.

## References
- [proj-prelude-module](proj-prelude-module.md)
- [proj-pub-crate-internal](proj-pub-crate-internal.md)
- [api-non-exhaustive](../../rust-coding-standards/rules/api-non-exhaustive.md)
