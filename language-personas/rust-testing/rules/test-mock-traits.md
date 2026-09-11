---
title: Depend on Traits, Not Concrete Types, to Enable Mocking
impact: HIGH
impactDescription: Turns database- or network-dependent tests into fast, deterministic in-memory ones
tags: [testing, mocking, traits, dependency-injection]
---

# Depend on Traits, Not Concrete Types, to Enable Mocking [HIGH]

## Description
A struct that holds a concrete `PostgresConnection` or `HttpClient` field can only be tested against a real Postgres instance or a real network call — there's no way to force a timeout, an error response, or a specific edge-case payload without actually reproducing that condition externally. Extracting the dependency behind a trait breaks that coupling: the production type implements the trait against the real service, a test-only type implements the same trait however the test needs it to behave, and the code under test is generic (or takes a trait object) over which one it gets. The result is that error paths, timeouts, and rare responses become as easy to test as the happy path, and the test suite no longer needs a live database to run.

## Bad Example
```rust
struct UserService {
    db: PostgresConnection, // concrete type — can't be tested without a real database
}

impl UserService {
    async fn get_user(&self, id: u64) -> Result<User, Error> {
        self.db.query("SELECT * FROM users WHERE id = $1", &[&id]).await
    }
}
```

## Good Example
```rust
#[async_trait]
trait UserRepository: Send + Sync {
    async fn find_by_id(&self, id: u64) -> Result<Option<User>, DbError>;
}

struct UserService<R: UserRepository> {
    repo: R,
}

impl<R: UserRepository> UserService<R> {
    async fn get_user(&self, id: u64) -> Result<User, Error> {
        self.repo.find_by_id(id).await?.ok_or(Error::NotFound)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use std::collections::HashMap;

    struct FakeRepo(HashMap<u64, User>);

    #[async_trait]
    impl UserRepository for FakeRepo {
        async fn find_by_id(&self, id: u64) -> Result<Option<User>, DbError> {
            Ok(self.0.get(&id).cloned())
        }
    }

    #[tokio::test]
    async fn get_user_returns_not_found_for_missing_id() {
        let service = UserService { repo: FakeRepo(HashMap::new()) };
        let result = service.get_user(999).await;
        assert!(matches!(result, Err(Error::NotFound)));
    }
}
```

## Notes
- A hand-written fake (a struct backed by a `HashMap`, as above) is often enough and keeps the trait's contract obvious; reach for a mocking crate like `mockall` when you need call-count or call-order assertions that a hand-written fake would be tedious to implement.
- A test double dedicated to failure (`struct FailingClient; impl HttpClient for FailingClient { ... always returns Err ... }`) makes timeout- and error-handling code trivially testable without needing to simulate the actual network condition.
- Making the service generic over the trait (`UserService<R: UserRepository>`) avoids any runtime cost; `Box<dyn UserRepository>` is the alternative when you'd rather not propagate a generic parameter through an entire call chain, at the cost of one vtable indirection per call.
- The trait boundary is also where async trait methods currently need `#[async_trait]` on stable Rust for object-safety and ergonomics — native `async fn` in traits has limitations here that the macro works around.

## References
- [api-sealed-trait](../../rust-coding-standards/rules/api-sealed-trait.md)
- [test-proptest-properties](test-proptest-properties.md)
- [proj-lib-main-split](../../rust-tooling/rules/proj-lib-main-split.md)
