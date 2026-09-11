---
title: Prefer Native async fn in Traits Over async-trait
impact: MEDIUM
impactDescription: Removes one heap allocation per call and the async-trait macro dependency
tags: [async, traits, afit, send]
---

# Prefer Native async fn in Traits Over async-trait [MEDIUM]

## Description
Since Rust 1.75, `async fn` can be written directly inside a trait definition without the `#[async_trait]` proc-macro. The macro's expansion boxes every returned future (`Pin<Box<dyn Future>>`), which costs a heap allocation on every call and pulls in a dependency purely to work around a language limitation that no longer exists. Native async fn in traits (AFIT) avoids both. It carries two caveats that decide when the macro is still the right tool: the native form is not yet object-safe, so `Box<dyn Trait>` doesn't compile against it, and the compiler-generated future is not `Send` by default, which blocks `tokio::spawn` on a multi-threaded runtime unless the return type is bounded explicitly.

## Bad Example
```rust
// Pulls in a proc-macro dependency and boxes every future on the heap,
// even for trait objects that are never actually needed.
use async_trait::async_trait;

#[async_trait]
trait Repo {
    async fn get(&self, id: u64) -> anyhow::Result<String>;
}

struct PgRepo;

#[async_trait]
impl Repo for PgRepo {
    async fn get(&self, id: u64) -> anyhow::Result<String> {
        Ok(format!("row-{id}"))
    }
}
```

## Good Example
```rust
// Native async fn in trait — no macro, no boxed future, for static dispatch.
trait Repo {
    async fn get(&self, id: u64) -> anyhow::Result<String>;
}

// Needs dyn dispatch or Send futures on a multi-threaded runtime:
// bound the return type explicitly instead of reaching for async-trait.
trait SendRepo {
    fn get(&self, id: u64) -> impl Future<Output = anyhow::Result<String>> + Send;
}

struct PgRepo;

impl Repo for PgRepo {
    async fn get(&self, id: u64) -> anyhow::Result<String> {
        Ok(format!("row-{id}"))
    }
}
```

## Notes
- Native AFIT is not dyn-compatible: `Box<dyn Repo>` fails to compile against the plain definition above. Keep `#[async_trait]`, or use the `trait-variant` crate's `#[trait_variant::make]`, which generates a boxed-future, dyn-compatible sibling trait alongside the native one.
- The compiler-generated future from a native async fn captures `&self` but makes no `Send` promise. Spawning it on a multi-threaded Tokio runtime fails to compile unless the trait method returns `impl Future<Output = T> + Send` explicitly, or the `trait-variant` `Send`-bounded variant is used.
- Default to native async fn for static dispatch (generics, `impl Trait` callers); reach for `#[async_trait]` or `trait-variant` only once a concrete need for `dyn Trait` or cross-thread spawning shows up.
- A single-threaded runtime (`current_thread` flavor or a `LocalSet`) never needs the `Send` bound, so native async fn works there with no extra ceremony.

## References
- [async-async-fn-bounds](async-async-fn-bounds.md)
- [async-tokio-runtime](async-tokio-runtime.md)
