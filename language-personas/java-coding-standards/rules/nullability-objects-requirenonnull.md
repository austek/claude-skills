---
title: Fail Fast with Objects.requireNonNull at API Boundaries
impact: HIGH
impactDescription: turns a distant NPE into an immediate, precisely-located one
tags: [nullability, validation, api-design]
---

# Fail Fast with Objects.requireNonNull at API Boundaries [HIGH]

## Description
Every public constructor and public method that does not accept `null` for a given parameter should validate that parameter with `Objects.requireNonNull(value, "message")` as its first action. Without this, a `null` argument travels silently into the object's state or the method body and surfaces as a `NullPointerException` far away — inside a stream pipeline, a hash map lookup, three calls deep — where the stack trace no longer points at the actual mistake. A boundary check turns that into an immediate, precisely-located failure naming the exact parameter.

This applies to constructors, public/protected method parameters, and static factory methods. It does not apply to private helper methods that only ever receive already-validated values — re-checking there is noise.

## Bad Example
```java
public class Order {
    private final Customer customer;
    private final List<LineItem> lineItems;

    public Order(Customer customer, List<LineItem> lineItems) {
        this.customer = customer;       // no check — null travels silently into state
        this.lineItems = lineItems;
    }

    public BigDecimal total() {
        return lineItems.stream()       // NPE here, far from the real mistake
            .map(LineItem::amount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

## Good Example
```java
public class Order {
    private final Customer customer;
    private final List<LineItem> lineItems;

    public Order(Customer customer, List<LineItem> lineItems) {
        this.customer = Objects.requireNonNull(customer, "customer");
        this.lineItems = List.copyOf(Objects.requireNonNull(lineItems, "lineItems"));
    }

    public BigDecimal total() {
        return lineItems.stream()
            .map(LineItem::amount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

// Constructor throws immediately, naming the bad argument
new Order(null, items); // NullPointerException: customer
```

## Notes
- Always pass the second `message` argument — the parameter name is enough; do not restate the value or the type.
- Combine with `List.copyOf` / `Map.copyOf` for collection parameters so the null check and the defensive copy happen in one line (see `immutability-defensive-copy`).
- `Objects.requireNonNullElse(value, default)` and `requireNonNullElseGet` cover the case where absence should fall back to a default instead of throwing.
- Do not use `assert` for this — assertions are disabled by default at runtime and are not a substitute for argument validation.

## References
- [Objects (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Objects.html)
- [Effective Java, 3rd Edition — Item 49: Check parameters for validity](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
