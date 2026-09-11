---
title: Use Bounded Wildcards at API Boundaries (PECS)
impact: MEDIUM
impactDescription: API accepts the widest correct set of caller types instead of rejecting valid callers
tags: [generics, wildcards, api-design, pecs]
---

# Use Bounded Wildcards at API Boundaries (PECS) [MEDIUM]

## Description
An API parameter typed as an exact generic (`List<Order>`) is more restrictive than necessary when the method only reads from or only writes into it — it rejects a caller holding `List<PriorityOrder>`, even though that list would work fine. PECS — "Producer Extends, Consumer Super" — is the rule for when to loosen the type with a bounded wildcard: if the parameter only produces values the method reads (`? extends T`), or only consumes values the method writes into it (`? super T`), a wildcard accepts every caller whose type is compatible with that direction, without losing any type safety. A parameter the method both reads from and writes to keeps its exact, unbounded type — no wildcard can express "both," and none is needed since the caller's type already matches exactly.

## Bad Example
```java
// Only ever reads from source, only ever writes into destination —
// but the exact types force callers to match precisely.
public static void copy(List<Order> source, List<Order> destination) {
    for (Order order : source) {
        destination.add(order);
    }
}

List<PriorityOrder> priorityOrders = List.of(new PriorityOrder("P-1"));
List<Order> allOrders = new ArrayList<>();
copy(priorityOrders, allOrders); // compile error: List<PriorityOrder> is not List<Order>
```

## Good Example
```java
// source only produces T (extends), destination only consumes T (super)
public static <T> void copy(List<? extends T> source, List<? super T> destination) {
    for (T item : source) {
        destination.add(item);
    }
}

List<PriorityOrder> priorityOrders = List.of(new PriorityOrder("P-1"));
List<Order> allOrders = new ArrayList<>();
copy(priorityOrders, allOrders); // compiles: PriorityOrder extends Order
```

## Notes
- A `? extends T` parameter cannot be written to (beyond `null`) — the compiler cannot know the runtime type is exactly compatible, so it forbids `add`.
- A `? super T` parameter can be read only as `Object` — the compiler only knows it accepts `T` or a supertype, not what it actually holds.
- This is a boundary concern: internal fields and return types generally stay unbounded generics, since the class itself needs the exact type to work with the value later.

## References
- [Effective Java, 3rd Edition — Item 31: Use bounded wildcards to increase API flexibility](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
