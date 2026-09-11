---
title: Return an Empty Collection, Never Optional of a Collection
impact: MEDIUM
impactDescription: removes a redundant absence wrapper around an already-absence-capable type
tags: [nullability, optional, collections, api-design]
---

# Return an Empty Collection, Never Optional of a Collection [MEDIUM]

## Description
A `List<T>`, `Set<T>`, or `Map<K, V>` is already a perfectly good representation of "nothing here" — an empty collection. Wrapping it in `Optional<List<T>>` gives the caller two ways to express the same absent state (`Optional.empty()` and `Optional.of(List.of())`), and forces every caller to unwrap before they can even ask "how many?". Return the collection directly and make the empty case the "not found" signal; never return `null` for a collection either — same rule, same reasoning as `nullability-optional-return`, just collection-shaped.

## Bad Example
```java
public class OrderRepository {
    public Optional<List<Order>> findByCustomerId(Long customerId) {
        List<Order> orders = queryOrders(customerId);
        return orders.isEmpty() ? Optional.empty() : Optional.of(orders);
        // two representations of "no orders": Optional.empty() and an empty list inside Optional.of
    }
}

// Caller must unwrap before it can even check size
int count = repository.findByCustomerId(id)
    .map(List::size)
    .orElse(0);
```

## Good Example
```java
public class OrderRepository {
    public List<Order> findByCustomerId(Long customerId) {
        return queryOrders(customerId); // empty list IS the "not found" signal
    }
}

// Caller uses the collection directly
List<Order> orders = repository.findByCustomerId(id);
int count = orders.size();
if (orders.isEmpty()) {
    log.info("No orders for customer {}", customerId);
}
```

## Notes
- This applies to arrays too: return an empty array (`new Order[0]`), never `null`, and never `Optional<Order[]>`.
- If the domain needs to distinguish "found zero results" from "the query itself failed", model that with `Either<Error, List<Order>>` (Vavr) or a checked exception — not by nesting `Optional` around the collection.
- `Collections.emptyList()` / `List.of()` return immutable singletons; prefer them over `new ArrayList<>()` when the empty result is never mutated by the caller.

## References
- [Effective Java, 3rd Edition — Item 54: Return empty collections or arrays, not nulls](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Effective Java, 3rd Edition — Item 55: Return optionals judiciously](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
