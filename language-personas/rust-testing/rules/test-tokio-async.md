---
title: Drive Async Tests with #[tokio::test]
impact: HIGH
impactDescription: Replaces manual runtime construction with a one-line attribute needed for any async test to run at all
tags: [testing, async, tokio, runtime]
---

# Drive Async Tests with #[tokio::test] [HIGH]

## Description
An `async fn` doesn't run on its own — it produces a `Future` that has to be polled by an executor, and plain `#[test]` has no idea how to do that, so an `async fn` marked `#[test]` simply fails to compile. `#[tokio::test]` solves this by expanding into the boilerplate you'd otherwise write by hand: constructing a Tokio runtime and calling `block_on` around the test body. Beyond convenience, it's configurable — you can pick a single-threaded runtime for determinism, control worker-thread count, or start the runtime with paused virtual time — which makes it the standard entry point for every async test in a Tokio-based codebase rather than a one-off convenience macro.

## Bad Example
```rust
// Compiles and runs, but re-derives by hand what the attribute gives for free,
// and is easy to get subtly wrong (forgetting to build a multi-thread runtime
// when the code under test actually spawns tasks).
#[test]
fn test_async_function() {
    let rt = tokio::runtime::Runtime::new().unwrap();
    rt.block_on(async {
        let result = fetch_data().await;
        assert!(result.is_ok());
    });
}
```

## Good Example
```rust
#[tokio::test]
async fn test_async_function() {
    let result = fetch_data().await;
    assert!(result.is_ok());
}

#[tokio::test]
async fn fetches_two_users_concurrently() {
    let (a, b) = tokio::join!(fetch_user(1), fetch_user(2));
    assert!(a.is_ok() && b.is_ok());
}
```

## Notes
- `#[tokio::test(flavor = "current_thread")]` runs the test on a single-threaded runtime, which is simpler and fully deterministic — prefer it unless the code under test genuinely depends on true parallelism (e.g. `tokio::spawn` racing against the test body).
- `#[tokio::test(start_paused = true)]` starts Tokio's virtual clock paused, so calling `tokio::time::advance(...)` lets you test timeout and delay logic in milliseconds of wall-clock time instead of actually waiting for real seconds to pass.
- `tokio::time::timeout(duration, future).await` wrapped around the operation under test is the standard way to assert both that something completes within a deadline (`assert!(result.is_ok())`) and that something correctly times out (`assert!(result.is_err())`).
- Testing an `mpsc` channel's behavior end-to-end — spawn a producer task, then assert on the sequence of values received, ending with `None` once the sender drops — is a good way to verify a channel-based protocol without mocking anything.

## References
- [async-tokio-runtime](../../rust-coding-standards/rules/async-tokio-runtime.md)
- [test-mock-traits](test-mock-traits.md)
- [test-fixture-raii](test-fixture-raii.md)
