---
title: Return the Interface Type, Not the Concrete Implementation
impact: HIGH
impactDescription: swapping the underlying implementation later stays a non-breaking change
tags: [api-design, encapsulation, interfaces]
---

# Return the Interface Type, Not the Concrete Implementation [HIGH]

## Description
A method signature returning `ArrayList<Order>` commits to that concrete class as part of the public contract, even when nothing about the method's purpose requires it — callers can now legally depend on `ArrayList`-specific behavior (index-based access performance, `ensureCapacity`), and switching the internal implementation to a different `List` later becomes a breaking change instead of a private refactor. Returning the interface (`List<Order>`) states only the contract the method actually promises, leaves every implementation detail free to change, and matches how the standard library itself is designed — factory methods like `List.of()` and `Collectors.toList()` return `List`, not a named concrete class. Apply the same principle to fields and parameters: depend on the interface unless a specific implementation's extra behavior is genuinely part of the contract.

## Bad Example
```java
public class OrderRepository {
    public ArrayList<Order> findByCustomer(String customerId) {
        ArrayList<Order> results = new ArrayList<>();
        // ... populate results
        return results; // callers can now depend on ArrayList specifically
    }
}

// Caller couples to the concrete type unnecessarily
ArrayList<Order> orders = repository.findByCustomer("C-1");
```

## Good Example
```java
public class OrderRepository {
    public List<Order> findByCustomer(String customerId) {
        List<Order> results = new ArrayList<>();
        // ... populate results
        return List.copyOf(results); // implementation detail stays internal
    }
}

// Caller depends only on the List contract
List<Order> orders = repository.findByCustomer("C-1");
```

## Notes
- This applies to the *declared* type at the API boundary — the method body is free to use whatever concrete implementation fits (`ArrayList`, `LinkedList`, `HashMap`) internally.
- The rare exception is when the extra behavior of a concrete type is genuinely the contract — returning `StringBuilder` because the caller needs to keep appending is different from returning it because that's what happened to be built internally.
- Combine with `List.copyOf`/`Map.copyOf` (or an unmodifiable wrapper) when the returned collection should not be mutable from outside — see `immutability-unmodifiable-collections`.

## References
- [Effective Java, 3rd Edition — Item 64: Refer to objects by their interfaces](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
