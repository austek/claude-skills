---
title: Wrap IDs in Newtypes Instead of Raw Integers
impact: HIGH
impactDescription: Turns ID mix-ups from a runtime bug into a compile error
tags: [type-safety, newtype, ids, api-design]
---

# Wrap IDs in Newtypes Instead of Raw Integers [HIGH]

## Description
Using a raw `u64` for every kind of identifier means the compiler can't stop a `user_id` from being passed where a `post_id` belongs — the two parameters are structurally identical, so a swapped argument order compiles fine and fails, if at all, at runtime. Wrapping each ID in its own single-field newtype makes the types distinct: `UserId` and `PostId` no longer unify, so swapping them is a compile error instead of a bug report. The wrapper costs nothing at runtime and is a natural place to hang `Display`, `From`, and serialization behavior specific to that ID.

## Bad Example
```rust
fn get_user_posts(user_id: u64, post_id: u64) -> Vec<Post> {
    todo!() // which is which? easy to swap by accident
}

// Compiles fine, wrong at runtime.
let posts = get_user_posts(post_id, user_id);
```

## Good Example
```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct UserId(pub u64);

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct PostId(pub u64);

fn get_user_posts(user_id: UserId, post_id: PostId) -> Vec<Post> {
    todo!()
}

// let posts = get_user_posts(post_id, user_id); // compile error: types don't match
let posts = get_user_posts(UserId(1), PostId(42));
```

## Notes
- Derive `Debug, Clone, Copy, PartialEq, Eq, Hash` as a baseline; add `PartialOrd, Ord` only if the ID needs sorting or a `BTreeMap` key.
- `#[serde(transparent)]` serializes the newtype as its bare inner value (`123`, not `{"0": 123}`), so JSON compatibility is unaffected.
- For string-backed IDs (UUIDs, slugs), validate at construction (`SessionId::parse`) rather than accepting an arbitrary `String`.
- A `macro_rules!` helper that defines the derive list and `From`/`new`/`get` once keeps many sibling ID types consistent without hand-duplicating boilerplate.

## References
- [api-newtype-safety](api-newtype-safety.md)
- [type-newtype-validated](type-newtype-validated.md)
- [api-parse-dont-validate](api-parse-dont-validate.md)
- [num-nonzero](num-nonzero.md)
