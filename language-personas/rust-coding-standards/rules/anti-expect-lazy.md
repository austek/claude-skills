---
title: Don't Reach for expect() Just Because It Has a Message
impact: HIGH
impactDescription: A custom message doesn't change a crash into a handled error
tags: [anti-pattern, error-handling, panics, code-smell]
---

# Don't Reach for expect() Just Because It Has a Message [HIGH]

## Description
`.expect("message")` feels more responsible than `.unwrap()` because it leaves a note explaining what happened — but the program still crashes either way. For a failure that can genuinely occur in production — a network timeout, a missing config file, a user typing garbage into a form — the custom message doesn't make the panic acceptable, it just makes the panic slightly easier to diagnose after the fact. That's a real but narrow benefit, and it's easy to mistake it for having actually handled the error. The fix is the same as for `.unwrap()`: propagate with `?`, or match explicitly and decide what the caller should see instead.

## Bad Example
```rust
async fn fetch_user(id: u64) -> User {
    // A network call and a lookup: both can fail in completely ordinary
    // production circumstances, not just as bugs.
    let response = client.get(url).await.expect("request should succeed");
    db.find_user(id).await.expect("user should exist")
}
```

## Good Example
```rust
async fn fetch_user(id: u64) -> Result<User, AppError> {
    let response = client.get(url).await.context("request failed")?;
    db.find_user(id).await?.ok_or(AppError::UserNotFound(id))
}
```

## Notes
- `.expect()` earns its place for a genuine invariant — a `Regex` compiled from a literal known valid at compile time, a mutex-poisoning path indicating a bug elsewhere in the program — where the message documents an assumption the code has already made true, not a possibility the caller needs to handle.
- The tell that separates the two cases: would a reasonable operator of this program ever see this message in production and think "yeah, that can happen sometimes"? If yes, it needs `Result`, not `.expect()`.
- `#![warn(clippy::expect_used)]` (stricter than `unwrap_used`, since it also flags the "safer-feeling" case) is worth enabling on any crate where `.expect()` has been used as a substitute for real error handling rather than for documenting a true invariant.

## References
- [err-expect-bugs-only](err-expect-bugs-only.md)
- [err-no-unwrap-prod](err-no-unwrap-prod.md)
- [anti-unwrap-abuse](anti-unwrap-abuse.md)
