---
title: Never Let an Error Vanish Into an Empty Match Arm
impact: HIGH
impactDescription: A silently discarded error is a bug with no trace left to debug from
tags: [anti-pattern, error-handling, code-smell]
---

# Never Let an Error Vanish Into an Empty Match Arm [HIGH]

## Description
`let _ = fallible_call();`, an `Err(_) => {}` match arm, and `.ok()` used to throw away a `Result` all share the same failure mode: whatever went wrong disappears with no trace, so the first sign of trouble is downstream — missing data, a state that should be impossible, a user report with no corresponding log line to explain it. The instinct behind writing code this way is usually reasonable ("this failure probably doesn't matter"), but that judgment call needs to be visible in the code, not silently baked into the absence of any handling at all. At minimum, log what happened; where the caller can act on it, propagate it instead.

## Bad Example
```rust
fn save_and_notify(record: Record) {
    let _ = database.save(record);      // did this even succeed?
    if let Err(_) = send_notification() {
        // nothing — failure is now invisible
    }
}
```

## Good Example
```rust
fn save_and_notify(record: Record) -> Result<(), AppError> {
    database.save(record)?; // propagate — caller decides what "failed" means here

    if let Err(e) = send_notification() {
        // Non-critical path: log instead of failing the whole operation,
        // but the failure is at least visible.
        tracing::warn!(error = ?e, "notification failed");
    }
    Ok(())
}
```

## Notes
- A batch operation that can't propagate a single failure without losing the rest of the batch should still collect failures and report a count or list at the end, rather than dropping each one individually as it occurs.
- The rare case where discarding really is correct (a `TcpStream::shutdown` error nobody can act on, a poisoned-mutex recovery path) deserves a one-line comment explaining why — `let _ = ...` with no comment looks identical whether the omission was a decision or an oversight.
- `clippy::let_underscore_drop` and `clippy::ignored_unit_patterns` catch some instances of this mechanically, but neither can judge whether a given discard was actually a reasoned decision — that judgment still needs a human reviewer or a comment.

## References
- [err-result-over-panic](err-result-over-panic.md)
- [err-context-chain](err-context-chain.md)
- [obs-error-chain](obs-error-chain.md)
