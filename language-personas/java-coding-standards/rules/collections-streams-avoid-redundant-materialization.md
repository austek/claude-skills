---
title: Avoid Redundant Materialization Between Stream Stages
impact: MEDIUM
impactDescription: removes an unnecessary full-collection allocation and a second traversal pass
tags: [streams, collections, performance]
---

# Avoid Redundant Materialization Between Stream Stages [MEDIUM]

## Description
Calling `.collect(Collectors.toList())` and then immediately looping or streaming over the result again to apply another transformation forces the data through a concrete `List` it never needed to become — an extra full allocation and an extra traversal pass, just to feed the next step. A `Stream` pipeline is lazy: chaining `.filter()`, `.map()`, and another `.filter()` before the single terminal `.collect()`/`.toList()` processes each element once, in one pass, with no intermediate collection materialized at all. Collect only at the boundary where a concrete collection is actually required — returning from the method, handing data to code that needs a `List`, or reusing the same stream contents more than once.

## Bad Example
```java
public List<String> topCustomerNames(List<Order> orders) {
    List<Order> largeOrders = orders.stream()
        .filter(o -> o.total().compareTo(BigDecimal.valueOf(1000)) > 0)
        .collect(Collectors.toList()); // materializes a full intermediate list

    List<String> names = new ArrayList<>();
    for (Order order : largeOrders) { // second pass over data already in hand
        names.add(order.customer().name());
    }
    return names;
}
```

## Good Example
```java
public List<String> topCustomerNames(List<Order> orders) {
    return orders.stream()
        .filter(o -> o.total().compareTo(BigDecimal.valueOf(1000)) > 0)
        .map(o -> o.customer().name())
        .toList(); // single pass, one materialization at the actual boundary
}
```

## Notes
- The same issue shows up as `.stream().collect(toList()).stream()...` — chaining a second `.stream()` call off a just-collected list; fold the operations into the original pipeline instead.
- An intermediate collection is legitimate when the data must be iterated more than once, sorted before further processing depends on that order, or handed to an API that only accepts a concrete `List`/`Set`.
- Profiling before optimizing still applies: for small, bounded collections this is a clarity issue more than a measurable performance one; for large or hot-path collections the extra pass and allocation are real costs.

## References
- [Stream (Java SE 21 & JDK 21) — laziness](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html)
- [Effective Java, 3rd Edition — Item 45: Use streams judiciously](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
