---
title: Keep Reflection Out of Business Logic
impact: HIGH
impactDescription: moves failures from runtime discovery back to compile time, and restores IDE navigation and static analysis
tags: [reflection, api-design, error-handling]
---

# Keep Reflection Out of Business Logic [HIGH]

## Description
Reflection (`Class.forName`, `Method.invoke`, field access via `Field.get`/`set`) trades away everything the compiler normally guarantees: a typo in a method name becomes a `NoSuchMethodException` at runtime instead of a compile error, a type mismatch becomes a `ClassCastException` at the call site instead of a type-checker rejection, and an IDE can no longer find callers of a method invoked only by name through `Method.invoke`. That cost is justified for framework and infrastructure code — a dependency-injection container, an ORM, a serialization library — that genuinely cannot know its target types at compile time. It is almost never justified inside business logic, where the types involved are known statically and reflection is usually standing in for a missing interface, a `switch` the author didn't want to write, or a plugin mechanism that a `sealed interface` would model more safely.

Where dynamic dispatch is genuinely needed inside application code, a `Map<String, Supplier<Handler>>` registry, a `sealed interface` with exhaustive `switch`, or standard dependency injection covers the same need with compile-time checking intact.

## Bad Example
```java
public Object applyDiscount(Order order, String strategyClassName) throws Exception {
    Class<?> strategyClass = Class.forName(strategyClassName);
    Object strategy = strategyClass.getDeclaredConstructor().newInstance();
    Method apply = strategyClass.getMethod("apply", Order.class);
    return apply.invoke(strategy, order); // any typo, signature drift, or missing class fails only at runtime
}
```

## Good Example
```java
public sealed interface DiscountStrategy permits PercentageDiscount, FlatDiscount {
    Order apply(Order order);
}

public Order applyDiscount(Order order, DiscountStrategy strategy) {
    return strategy.apply(order); // compiler verifies the call; exhaustiveness verified at the switch that builds strategy
}
```

## Notes
- Reflective access into a module's internals also fights the module system — see `modules-jpms-explicit-exports`; an unexported package blocks reflection from outside the module unless the module explicitly `opens` it.
- `instanceof` chains dispatching on runtime type are a milder version of the same problem — prefer `sealed-pattern-no-instanceof-chains`.
- Where a framework's use of reflection is unavoidable (JPA entities, Jackson DTOs), keep it confined to boundary types, not domain/business-logic classes — see `reflection-serialization-jackson-explicit-config`.

## References
- [java.lang.reflect package specification](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/package-summary.html)
- [Effective Java, 3rd Edition — Item 65: Prefer interfaces to reflection](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
