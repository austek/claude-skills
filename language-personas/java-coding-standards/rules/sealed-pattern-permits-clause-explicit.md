---
title: Write the permits Clause Explicitly When It Aids Readability
impact: LOW
impactDescription: a hierarchy's full variant list is visible at the declaration, not scattered across files
tags: [sealed-classes, readability, api-design]
---

# Write the permits Clause Explicitly When It Aids Readability [LOW]

## Description
Java lets `permits` be omitted when every permitted subtype is declared in the same file as the sealed type — the compiler infers the clause from the file's contents. That implicit form is fine for a small, single-file hierarchy (a sealed interface with its record implementations nested directly inside it, as in `sealed-pattern-sealed-interface-over-enum-hierarchy`). Once the permitted types live in separate files, or the hierarchy is large enough that seeing the full variant list matters for understanding it, write `permits` explicitly: it turns "which types implement this" from something a reader has to search the package for into something stated at the declaration itself, and it is what the compiler checks against regardless of whether it is written out or inferred.

## Bad Example
```java
// Order.java — permits omitted; a reader must search the package
// for every file that implements OrderStatus to know the full variant set.
public sealed interface OrderStatus {}

// Pending.java, Shipped.java, Cancelled.java, Delivered.java — four separate files
public record Pending(Instant placedAt) implements OrderStatus {}
```

## Good Example
```java
// Order.java — the full variant set is visible right here, even though
// each record lives in its own file.
public sealed interface OrderStatus
    permits Pending, Shipped, Cancelled, Delivered {}

// Pending.java
public record Pending(Instant placedAt) implements OrderStatus {}
```

## Notes
- The implicit, same-file form is a legitimate default for small hierarchies — this rule is about readability at scale, not a blanket requirement to always write `permits`.
- `permits` cannot list a type outside the sealed type's module (or package, for an unnamed module) — cross-file, same-package implementations are the case where writing it explicitly pays off most.
- Keep the `permits` list and the actual implementing files in sync manually when split across files; the compiler still enforces consistency, but nothing keeps the declaration order matching file layout.

## References
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
