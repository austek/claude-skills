---
title: Use Option for Values That Might Not Exist
impact: CRITICAL
impactDescription: Eliminates null-pointer-style bugs by forcing the absent case to be handled
tags: [type-safety, option, null-safety, api-design]
---

# Use Option for Values That Might Not Exist [CRITICAL]

## Description
`Option<T>` puts "value or nothing" directly into the type system instead of relying on a sentinel value, an "empty" placeholder object, or a null pointer that carries no compiler guarantee. A function returning a sentinel — an empty `User` struct meaning "not found" — lets a caller use it as if it were real data, with the bug surfacing wherever that placeholder's fields get read downstream. `Option<T>` closes that path entirely: there is no way to reach the `T` without going through `Some`/`None`, so a caller who forgets to check absence gets a compile error, not a subtle runtime defect.

## Bad Example
```rust
// Sentinel value: caller might not check, and the "empty" user looks real.
fn find_user(id: u64) -> User {
    users.get(&id).cloned().unwrap_or(User::empty())
}

let user = find_user(42);
println!("{}", user.name); // might be the empty sentinel -- silent bug
```

## Good Example
```rust
fn find_user(id: u64) -> Option<User> {
    users.get(&id).cloned()
}

match find_user(42) {
    Some(u) => println!("{}", u.name),
    None => println!("user not found"),
}

// Or with combinators:
let name = find_user(42).map(|u| u.name).unwrap_or_else(|| "unknown".to_string());
```

## Notes
- `?` on an `Option`-returning function propagates `None` the same way it propagates `Err` on `Result` — chain fallible lookups with `.and_then(...)` or early `?`.
- `Option<&T>` for optional borrows, `.as_deref()` to go from `Option<String>` to `Option<&str>` without cloning.
- `Option` communicates "might not exist" with no error context; `Result` communicates "might have failed" with a reason — pick based on whether the caller needs to know *why*.
- `.ok_or(Error::NotFound)?` converts an `Option` into a `Result` at the boundary where absence should actually become an error.

## References
- [type-result-fallible](type-result-fallible.md)
- [type-enum-states](type-enum-states.md)
- [err-no-unwrap-prod](err-no-unwrap-prod.md)
