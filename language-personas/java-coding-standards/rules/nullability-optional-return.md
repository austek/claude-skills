---
title: Return Optional Instead of Null for Absent Values
impact: HIGH
impactDescription: eliminates NullPointerException at call sites
tags: [nullability, optional, api-design]
---

# Return Optional Instead of Null for Absent Values [HIGH]

## Description
A method whose return value may legitimately be absent should return `Optional<T>`, never `null`. A `null` return forces every caller to remember, without compiler help, that this particular method is one of the ones that can hand back nothing. `Optional<T>` puts that fact in the signature, so the compiler and the IDE carry the reminder instead of the caller's memory. It also gives the caller a small, composable API (`map`, `filter`, `orElseGet`, `orElseThrow`) instead of an `if (x != null)` branch repeated at every call site.

This rule covers return values only. Never use `Optional` for a field or a method parameter — see `nullability-no-optional-field` and `nullability-no-optional-param`.

## Bad Example
```java
public class UserRepository {
    private final Map<Long, User> usersById;

    public User findById(Long id) {
        return usersById.get(id); // returns null when absent — silent trap for callers
    }
}

// Caller has no signal that null is possible
User user = repository.findById(id);
sendWelcomeEmail(user.getEmail()); // NullPointerException when id is unknown
```

## Good Example
```java
public class UserRepository {
    private final Map<Long, User> usersById;

    public Optional<User> findById(Long id) {
        return Optional.ofNullable(usersById.get(id));
    }
}

// Caller is forced to decide the absent case
repository.findById(id)
    .map(User::getEmail)
    .ifPresentOrElse(
        this::sendWelcomeEmail,
        () -> log.warn("No user found for id {}", id));
```

## Notes
- Wrap a value that may be `null` with `Optional.ofNullable`, never `Optional.of` — `Optional.of(null)` throws `NullPointerException` immediately.
- `Optional<T>` is not `Serializable` and carries wrapper overhead; keep it at API boundaries and return types, not inside high-throughput internal loops.
- Prefer `orElseThrow(...)` over `get()` so a missing value fails with a meaningful exception, not a generic `NoSuchElementException`.
- A method returning a collection should never wrap it in `Optional` — return an empty collection instead (see `nullability-no-optional-collection`).

## References
- [Effective Java, 3rd Edition — Item 55: Return optionals judiciously](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Optional (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html)
