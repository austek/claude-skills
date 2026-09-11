---
title: Never Give a Record a Mutable Component or Backing Field
impact: HIGH
impactDescription: preserves the immutability guarantee that is the entire point of choosing a record
tags: [records, immutability, encapsulation]
---

# Never Give a Record a Mutable Component or Backing Field [HIGH]

## Description
A record's components are implicitly `final`, but `final` only prevents reassigning the reference — it does nothing to stop a mutable object held by that reference (a `List`, a `Date`, an array, a mutable custom class) from being changed after construction through its own methods. A record holding such a reference directly is only shallowly immutable: `equals`/`hashCode` computed at one point can silently go stale, and code elsewhere holding the same record can observe it changing out from under them, which defeats the reason to reach for a record over a class in the first place (`immutability-records-for-data`). Store an immutable type instead (`List<T>` built with `List.copyOf`, an immutable value type), or defensively copy a mutable argument in the compact constructor before it's assigned.

## Bad Example
```java
public record Order(String id, List<LineItem> items) {}

List<LineItem> items = new ArrayList<>();
items.add(new LineItem("SKU-1", 2));
Order order = new Order("O-1", items);

items.add(new LineItem("SKU-2", 5)); // mutates the caller's list...
order.items().size(); // ...and order.items() now returns 2 items — the record changed after construction
```

## Good Example
```java
public record Order(String id, List<LineItem> items) {
    public Order {
        items = List.copyOf(items); // defensive copy: order.items() can never change after this
    }
}

List<LineItem> items = new ArrayList<>();
items.add(new LineItem("SKU-1", 2));
Order order = new Order("O-1", items);

items.add(new LineItem("SKU-2", 5)); // does not affect order — it copied its own snapshot
order.items().size(); // still 1
```

## Notes
- `List.copyOf` throws on a `null` element and produces a genuinely unmodifiable list — prefer it over `Collections.unmodifiableList`, which only wraps the original (still-mutable) list rather than copying it.
- The same applies to a mutable custom type as a component: either make that type itself immutable, or copy it (a defensive-copy constructor or a `copy()` method) in the compact constructor.
- `LineItem` in the example must itself be immutable (a record, most naturally) for the guarantee to hold all the way down — a `List.copyOf` of mutable elements still leaves each element mutable.

## References
- [Effective Java, 3rd Edition — Item 50: Make defensive copies when needed](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [JEP 395: Records](https://openjdk.org/jeps/395)
