---
title: Never Expose Mutable Static State
impact: CRITICAL
impactDescription: eliminates a class of test-order-dependent failures and unsynchronized cross-thread races at once
tags: [anti-patterns, concurrency, immutability, testability]
---

# Never Expose Mutable Static State [CRITICAL]

## Description
A mutable `static` field is shared by every thread and every caller in the JVM for the lifetime of the class — there is no instance boundary limiting who can read or write it, which makes it simultaneously a concurrency hazard (concurrent writers need explicit synchronization the field's declaration doesn't advertise or enforce) and a testability hazard (tests that mutate it leak state into whichever test happens to run next, producing failures that depend on execution order and vanish when a single test is run in isolation — one of the most time-consuming categories of flaky test to diagnose). A static mutable counter, cache, or "current context" field is usually reaching for global, ambient state as a shortcut around passing a value explicitly or injecting a proper collaborator.

A `static final` field holding an immutable value (a constant, an immutable collection built once via `List.of()`) is fine — it's the mutability, not the `static` keyword itself, that causes the problem. Where shared mutable state is genuinely required (a process-wide cache, a counter), it belongs behind an explicitly synchronized or concurrent-collection-backed class, instantiated once and injected like any other collaborator, not exposed as a bare static field.

## Bad Example
```java
public class RequestContext {
    public static String currentUserId; // shared by every thread; any test that sets it leaks into the next
}

// thread A:
RequestContext.currentUserId = "alice";
// thread B, concurrently:
RequestContext.currentUserId = "bob";
// thread A now reads "bob" — a data race with no compiler warning
```

## Good Example
```java
public final class RequestContext {
    private static final ThreadLocal<String> currentUserId = new ThreadLocal<>();

    public static void set(String userId) { currentUserId.set(userId); }
    public static String get() { return currentUserId.get(); }
    public static void clear() { currentUserId.remove(); } // called at request end to avoid leaking into thread-pool reuse
}
```
Better still, pass the user id explicitly through the call chain or inject a request-scoped bean where the framework supports it — `ThreadLocal` avoids the cross-thread race but still carries implicit, hard-to-trace state.

## Notes
- `ThreadLocal` must be cleared at the end of each logical unit of work (a request, a task) when running on a pooled thread — otherwise state leaks into the next task that reuses the same thread.
- See `oo-design-dependency-injection-over-static-singletons` for the broader case against static, shared instances in place of injected collaborators.
- A static field that's mutated only once, during class initialization, and never again is effectively immutable and not what this rule targets — the risk is state that changes during normal operation.

## References
- [Effective Java, 3rd Edition — Item 17: Minimize mutability](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Java Concurrency in Practice — Chapter 3: Sharing Objects](https://jcip.net/)
