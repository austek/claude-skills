---
title: No Mutation Inside Stream Operations
impact: HIGH
impactDescription: avoids undefined behavior under parallel execution and non-obvious ordering bugs
tags: [streams, side-effects, functional, thread-safety]
---

# No Mutation Inside Stream Operations [HIGH]

## Description
Stream operations — the lambda passed to `map`, `filter`, `peek`, or `forEach` — are meant to be stateless and free of side effects on shared mutable state. Using `forEach` to push results into an external `List` or accumulator, or mutating a captured variable from inside `map`, works by accident under sequential execution but is explicitly unspecified behavior per the `Stream` API contract, and breaks — silently, without a compile error, sometimes without even an exception — the moment the stream becomes `.parallel()` or an unrelated refactor introduces parallelism. It also defeats the readability benefit of streams in the first place: a `forEach` full of mutation is a `for` loop wearing a stream's syntax. Collect into a result with `collect`/`toList`/`reduce`, and reserve `forEach` for genuine, single terminal side effects like logging or writing to an external system.

## Bad Example
```java
public List<String> upperCaseNames(List<User> users) {
    List<String> names = new ArrayList<>(); // captured, mutated from inside forEach
    users.stream()
        .filter(User::isActive)
        .forEach(user -> names.add(user.getName().toUpperCase())); // unspecified under .parallel()
    return names;
}
```

## Good Example
```java
public List<String> upperCaseNames(List<User> users) {
    return users.stream()
        .filter(User::isActive)
        .map(User::getName)
        .map(String::toUpperCase)
        .toList(); // no shared mutable state, safe under sequential or parallel execution
}

// forEach is fine for a genuine terminal side effect with no result to collect
public void notifyActiveUsers(List<User> users) {
    users.stream()
        .filter(User::isActive)
        .forEach(notificationService::send);
}
```

## Notes
- `Stream.peek` is for debugging only (e.g., logging elements mid-pipeline); it must never be used to perform the pipeline's actual work, since its execution is not guaranteed for every element under all terminal operations.
- The rule extends to `map`/`filter` lambdas: they must not mutate their input argument or any field outside their own scope — a `map` lambda should return a new value, not mutate and return the same reference.
- This is why `collections-streams-prefer-stream-pipeline` exists: the correct fix for "I need to accumulate results" is a `collect`/`toList` terminal operation, not a mutated external collection inside `forEach`.

## References
- [Stream (Java SE 21 & JDK 21) — Stateless behaviors](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html)
- [Effective Java, 3rd Edition — Item 46: Prefer side-effect-free functions in streams](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
