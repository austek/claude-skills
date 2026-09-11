---
title: Switch Exhaustively Over Sealed Types — No default Branch
impact: HIGH
impactDescription: adding a new variant becomes a compile error at every switch that must handle it
tags: [sealed-classes, switch, pattern-matching, java21, exhaustiveness]
---

# Switch Exhaustively Over Sealed Types — No default Branch [HIGH]

## Description
When a `switch` (Java 21+ pattern matching for switch, JEP 441) covers every `permits` implementation of a sealed type, the compiler verifies exhaustiveness itself and the switch needs no `default` branch. This is the entire payoff of sealing a hierarchy: add a new variant to the `permits` clause, and every switch over that type that does not yet handle it fails to compile, at the exact call site that needs updating — not at runtime, and not only where a test happens to exercise the new case. Adding a `default` branch throws this guarantee away silently; the switch keeps compiling after a new variant is added, quietly routing it into whatever the `default` does, which is often wrong for the new case and never flagged by the compiler.

## Bad Example
```java
sealed interface Shape permits Circle, Rectangle, Triangle {}

double area(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        default -> 0; // Triangle silently falls here — and so will any future variant
    };
}
```

## Good Example
```java
sealed interface Shape permits Circle, Rectangle, Triangle {}

double area(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        case Triangle t -> 0.5 * t.base() * t.height();
        // No default: compiler verifies this covers every permitted Shape.
        // Adding a new permits implementation breaks this switch until handled.
    };
}
```

## Notes
- Exhaustiveness checking requires the switch to be over the sealed type itself (or an interface all of whose permitted subtypes are covered) — switching over `Object` or an unrelated supertype does not get this guarantee.
- A `default` branch is still appropriate for a genuinely non-sealed, open-ended type (an `int`, a `String`, an unsealed interface) — the rule is specific to closed, sealed hierarchies where exhaustiveness is knowable.
- `sealed-pattern-record-patterns` extends this same exhaustiveness checking into destructuring the record components of each variant, not just discriminating between them.

## References
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
