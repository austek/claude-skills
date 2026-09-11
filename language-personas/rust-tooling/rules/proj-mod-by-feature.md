---
title: Organize Modules by Feature, Not by Type
impact: MEDIUM
impactDescription: Confines a feature change to one directory instead of four
tags: [tooling, project-structure, modules, organization]
---

# Organize Modules by Feature, Not by Type [MEDIUM]

## Description
Splitting a codebase into `controllers/`, `models/`, `services/`, `repositories/` groups files by what technical role they play, which sounds tidy but scatters everything related to a single feature — say, orders — across four different directories that have to be found and opened separately just to understand how orders work end to end. Grouping by feature instead (`order/model.rs`, `order/repository.rs`, `order/service.rs`, `order/handler.rs`, all under one `order/` directory) keeps everything one feature needs in one place: understanding, modifying, or deleting a feature becomes a single-directory operation instead of a hunt across the whole type-based tree. This also makes feature ownership legible at a glance — a team or contributor working on orders touches one folder, not one file in each of four.

## Bad Example
```
src/
├── controllers/
│   ├── user_controller.rs
│   └── order_controller.rs
├── models/
│   ├── user.rs
│   └── order.rs
├── services/
│   ├── user_service.rs
│   └── order_service.rs
└── repositories/
    ├── user_repository.rs
    └── order_repository.rs
```

## Good Example
```
src/
├── user/
│   ├── mod.rs           # re-exports the public surface
│   ├── model.rs
│   ├── repository.rs
│   └── service.rs
├── order/
│   ├── mod.rs
│   ├── model.rs
│   ├── repository.rs
│   └── service.rs
└── lib.rs
```

```rust
// src/user/mod.rs
mod model;
mod repository;
mod service;

pub use model::{User, UserId};
pub(crate) use service::UserService; // internal-only
```

## Notes
- Deleting a feature under this scheme is a single `rm -rf src/order/` — under type-based grouping, removing a feature means hunting through four separate folders for every file that mentions it, and hoping nothing gets missed.
- Code genuinely shared across multiple features (a database connection pool, a common error type, auth middleware) belongs in its own `shared/` module rather than being force-fit into any one feature's directory.
- For a feature complex enough to need further breakdown, nest by concern inside the feature folder rather than flattening: `billing/invoice/` and `billing/payment/` each getting their own `model.rs` and `service.rs` keeps the hybrid structure scalable without abandoning the feature-first organization.
- Small features don't need every sub-file the pattern suggests — a `user/` folder with just `mod.rs` (containing the struct directly) and `repository.rs` is fine; add `service.rs` or `handler.rs` only once there's enough logic to justify separating it out.

## References
- [proj-flat-small](proj-flat-small.md)
- [proj-pub-use-reexport](proj-pub-use-reexport.md)
- [proj-lib-main-split](proj-lib-main-split.md)
