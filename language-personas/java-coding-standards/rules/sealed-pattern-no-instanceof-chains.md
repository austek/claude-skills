---
title: Replace instanceof Chains with Exhaustive Switch Over a Sealed Type
impact: HIGH
impactDescription: turns a silently-wrong fallthrough branch into a compile error at every call site
tags: [sealed-classes, instanceof, switch, code-quality]
---

# Replace instanceof Chains with Exhaustive Switch Over a Sealed Type [HIGH]

## Description
A chain of `if (x instanceof A) ... else if (x instanceof B) ... else ...` has no compiler-enforced connection to the actual set of possible types `x` can hold — nothing catches a missing branch, and nothing fails when a new type is added to the hierarchy, until it hits the `else` (or falls through with the wrong behavior) at runtime. Once the type in question is a sealed hierarchy, a `switch` with pattern matching (`sealed-pattern-switch-exhaustiveness`) replaces the chain entirely and turns that same gap into a compile error: the compiler knows every `permits` implementation and refuses to compile a switch that doesn't handle all of them. This is the concrete payoff of sealing a hierarchy — it is worth doing specifically so call sites can make this trade.

## Bad Example
```java
String describe(Shape shape) {
    if (shape instanceof Circle c) {
        return "circle r=" + c.radius();
    } else if (shape instanceof Rectangle r) {
        return "rect " + r.width() + "x" + r.height();
    }
    // Triangle added to the sealed hierarchy later: this chain still compiles,
    // silently returns null for it, and nothing flags the gap.
    return null;
}
```

## Good Example
```java
String describe(Shape shape) {
    return switch (shape) {
        case Circle c -> "circle r=" + c.radius();
        case Rectangle r -> "rect " + r.width() + "x" + r.height();
        case Triangle t -> "triangle b=" + t.base();
        // Adding a new Shape implementation breaks this switch until updated.
    };
}
```

## Notes
- This applies specifically once the type is sealed; an `instanceof` chain over a genuinely open, non-sealed hierarchy has no exhaustive alternative, since there is no fixed set of subtypes to check against.
- Pattern matching `instanceof` (Java 16+, `if (shape instanceof Circle c)`) already removes the manual cast that older `instanceof` chains needed — the remaining problem this rule addresses is the missing exhaustiveness check, which only a switch over a sealed type provides.
- A single-type `instanceof` check (no chain, no sealed hierarchy involved) is not what this rule targets — it applies once branching on "which subtype is this" becomes multi-way.

## References
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441)
- [JEP 394: Pattern Matching for instanceof](https://openjdk.org/jeps/394)
