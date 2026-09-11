---
title: Make Fields Final by Default
impact: HIGH
impactDescription: eliminates a whole class of mutation and reordering bugs
tags: [immutability, thread-safety, api-design]
---

# Make Fields Final by Default [HIGH]

## Description
Every field should be `final` unless there is a specific, stated reason it must vary after construction. A `final` field is assigned exactly once, and the compiler enforces that — it can only be set in the constructor (or an initializer), so any code that changes an object's field values later is caught at compile time, not discovered in production. `final` fields also let the JVM safely publish a fully-constructed object across threads without extra synchronization, as long as the constructor does not leak `this`. Mutable fields should be the exception you reach for deliberately (a cache, a counter, a builder under construction), not the default shape of a class.

## Bad Example
```java
public class ShoppingCart {
    private List<LineItem> items;   // mutable reference, reassignable from anywhere
    private BigDecimal total;       // recomputed and reassigned in multiple methods

    public ShoppingCart(List<LineItem> items) {
        this.items = items;
        this.total = computeTotal(items);
    }

    public void addItem(LineItem item) {
        items.add(item);            // mutates shared state
        total = computeTotal(items); // easy to forget this line elsewhere and desync
    }
}
```

## Good Example
```java
public final class ShoppingCart {
    private final List<LineItem> items;

    public ShoppingCart(List<LineItem> items) {
        this.items = List.copyOf(items);
    }

    public ShoppingCart withItem(LineItem item) {
        List<LineItem> updated = new ArrayList<>(items);
        updated.add(item);
        return new ShoppingCart(updated); // returns a new cart, never mutates this one
    }

    public BigDecimal total() {
        return items.stream()
            .map(LineItem::amount)
            .reduce(BigDecimal.ZERO, BigDecimal::add); // derived on demand, never desyncs
    }
}
```

## Notes
- `final` on a field only prevents reassigning the reference; a `final List<LineItem>` still needs `List.copyOf` or an unmodifiable wrapper to stop the referenced collection itself from being mutated (see `immutability-unmodifiable-collections`).
- Prefer records over hand-written immutable classes when the type is a pure data carrier — records make every component `final` and generate the accessors, `equals`, `hashCode`, and `toString` (see `immutability-records-for-data`).
- A field that genuinely must change (a mutable cache field, a lazily-initialized value) should be `private` and documented with why it is not `final`, and should be reviewed for thread-safety separately.

## References
- [Java Language Specification — 17.5 final Field Semantics](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.5)
- [Effective Java, 3rd Edition — Item 17: Minimize mutability](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
