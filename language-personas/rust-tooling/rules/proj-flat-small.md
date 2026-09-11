---
title: Keep Small Projects Flat
impact: LOW
impactDescription: Removes navigation overhead a project this size gets no benefit from paying
tags: [tooling, project-structure, simplicity]
---

# Keep Small Projects Flat [LOW]

## Description
Directory structure is supposed to pay for itself by making related code easier to find, but for a project with a handful of files, nested folders do the opposite — you end up clicking through `core/domain/models/user.rs` to reach fifty lines of code that could have just been `user.rs` next to everything else. A flat `src/` with `main.rs`, `config.rs`, `database.rs`, `user.rs` sitting side by side is faster to scan than the equivalent split across `core/`, `domain/`, `infrastructure/`, and `application/` folders, each holding one or two files and a `mod.rs` that does nothing but re-export. Structure should be a response to actual complexity — files growing past a few hundred lines, or a feature accumulating enough pieces that grouping them clarifies rather than obscures — not something applied upfront because it looks more "enterprise."

## Bad Example
```
src/
├── core/
│   └── mod.rs           # just re-exports
├── domain/
│   └── models/
│       └── user.rs      # 50 lines, three directories deep
├── infrastructure/
│   └── database/
│       └── connection.rs # 30 lines
└── main.rs
```

## Good Example
```
src/
├── main.rs
├── lib.rs
├── config.rs
├── database.rs
├── user.rs
└── error.rs
```

## Notes
- A rough sizing guide: under about 10 files, stay flat; 10-20 files, start grouping by feature; past 20, feature folders with their own submodules start earning their keep.
- Watch for the tell that a project has been over-structured before it needed to be: folders holding only one or two files, `mod.rs` files whose entire content is re-exports, or a module declaration section longer than the actual code it wraps.
- Conversely, files exceeding a few hundred lines, related code that's hard to locate, or filenames leaning on `_` prefixes for grouping (`user_model.rs`, `user_service.rs` sitting flat instead of nested under `user/`) are the actual signals it's time to add structure — not a fixed file count reached in advance.
- Growing a project in stages works better than deciding the final shape upfront: start flat, promote a cluster of related files (`order.rs`, `order_item.rs`) into `order/` only once that cluster is genuinely complex enough that the extra directory pays for itself.

## References
- [proj-mod-by-feature](proj-mod-by-feature.md)
- [proj-lib-main-split](proj-lib-main-split.md)
- [proj-mod-rs-dir](proj-mod-rs-dir.md)
