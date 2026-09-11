---
title: Document the Contract, Not the Implementation, on Public APIs
impact: HIGH
impactDescription: lets callers use a method correctly from its signature alone, without reading the body
tags: [javadoc, api-design, documentation]
---

# Document the Contract, Not the Implementation, on Public APIs [HIGH]

## Description
Every public class, interface, and method a caller outside its own package can reach needs a Javadoc comment that states its contract: what it does, what inputs are valid, what it returns, what it throws, and under what conditions — not how it's currently implemented. A caller consumes the contract; the implementation is free to change on the next commit, and a contract phrased around implementation detail ("iterates the internal list and returns the first match") breaks as soon as that detail does, even though the method's actual behavior toward callers hasn't changed at all. Writing the contract instead ("returns the first element matching the predicate, or empty if none match") stays accurate across any implementation that preserves the behavior.

This is also where invariants that aren't visible in the signature belong: whether `null` is accepted, whether the method is safe to call concurrently, whether it mutates its argument. A signature alone can't express these; the Javadoc is the only place a caller can learn them without reading source.

## Bad Example
```java
/**
 * Loops through the orders list and sums the totals field using a for loop.
 */
public BigDecimal totalValue(List<Order> orders) {
    BigDecimal total = BigDecimal.ZERO;
    for (Order order : orders) {
        total = total.add(order.total());
    }
    return total;
}
```

## Good Example
```java
/**
 * Returns the sum of {@link Order#total()} across all given orders.
 *
 * @param orders the orders to total; must not be {@code null} or contain {@code null}
 * @return the combined total, or {@link BigDecimal#ZERO} if {@code orders} is empty
 */
public BigDecimal totalValue(List<Order> orders) {
    return orders.stream().map(Order::total).reduce(BigDecimal.ZERO, BigDecimal::add);
}
```

## Notes
- Package-private and private members don't need Javadoc for external callers, but a non-obvious contract is still worth documenting for maintainers inside the package.
- State nullability and thread-safety explicitly rather than leaving them implicit — `nullability-objects-requirenonnull` and `concurrency-thread-safety-documentation` cover the mechanics for each.
- A contract that's hard to state clearly is often a sign the method is doing too much — see `oo-design-single-responsibility`.

## References
- [How to Write Doc Comments for the Javadoc Tool (Oracle)](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)
- [Effective Java, 3rd Edition — Item 56: Write doc comments for all exposed API elements](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
