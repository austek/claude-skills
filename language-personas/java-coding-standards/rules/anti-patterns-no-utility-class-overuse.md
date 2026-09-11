---
title: Don't Default to Static Utility Classes for Behavior That Belongs on a Type
impact: MEDIUM
impactDescription: keeps behavior next to the data it operates on instead of scattering it across unrelated Utils classes
tags: [anti-patterns, oo-design, api-design]
---

# Don't Default to Static Utility Classes for Behavior That Belongs on a Type [MEDIUM]

## Description
A genuine stateless utility class — `java.util.Collections`, `java.nio.file.Files`, a small class of pure helper functions with no natural owning type — is a legitimate, idiomatic pattern. The anti-pattern is reaching for a growing `Utils`/`Helper` class as the default home for any new piece of logic, rather than asking whether the behavior actually belongs as a method on an existing domain type. `OrderUtils.calculateTotal(order)` taking an `Order` as its first argument is a strong signal the method belongs on `Order` itself (or on a small, purpose-built collaborator, per `anti-patterns-no-god-classes`) — static-dispatched procedural code operating on someone else's data is exactly the shape object orientation exists to replace, and it can't be overridden, mocked through an interface, or discovered via the type it operates on the way an instance method can.

A `Utils` class also has no natural boundary — because there's no cohesive concept forcing new methods to relate to existing ones, it accretes unrelated behavior indefinitely, becoming the general-purpose miscellany that `anti-patterns-no-god-classes` addresses at the class-cohesion level.

## Bad Example
```java
public final class OrderUtils {
    private OrderUtils() {}

    public static Money calculateTotal(Order order) { /* uses order's line items */ }
    public static boolean isEligibleForFreeShipping(Order order) { /* uses order's weight, total */ }
    public static String formatForInvoice(Order order) { /* uses order's fields */ }
}

// every call site couples to a static, non-overridable procedure operating on Order's data from outside it
Money total = OrderUtils.calculateTotal(order);
```

## Good Example
```java
public final class Order {
    private final List<LineItem> items;

    public Money total() { /* uses this.items directly */ }
    public boolean isEligibleForFreeShipping() { /* uses this.weight(), this.total() */ }
    public String formatForInvoice() { /* uses this's own fields */ }
}

Money total = order.total(); // discoverable via Order's own API, testable, overridable if Order becomes an interface
```

## Notes
- A genuine cross-cutting helper with no single owning type — `Collections.unmodifiableList`, a generic `Comparator` combinator — is exactly what a static utility class is for; the test is whether the first parameter's type is the obvious sole owner of the behavior.
- Where behavior needs to vary by implementation rather than live on one concrete type, a `sealed interface` with the method declared on it is usually a better fit than either a utility class or a single mutable class with a type-tag field — see `sealed-pattern-sealed-interface-over-enum-hierarchy`.
- Utility classes are also a common home for reflection-based helpers reaching into another type's internals — see `reflection-serialization-avoid-for-business-logic` for why that's a related but distinct problem.

## References
- [Effective Java, 3rd Edition — Item 4: Enforce noninstantiability with a private constructor](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Refactoring, 2nd Edition — Martin Fowler, "Move Method"](https://martinfowler.com/books/refactoring.html)
