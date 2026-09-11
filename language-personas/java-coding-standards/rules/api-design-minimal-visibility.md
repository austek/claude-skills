---
title: Minimize the Visibility of Every Class and Member
impact: HIGH
impactDescription: shrinks the API surface callers can depend on, so internals stay free to change
tags: [api-design, encapsulation, visibility]
---

# Minimize the Visibility of Every Class and Member [HIGH]

## Description
Every `public` class, method, or field is a permanent commitment — once an external caller depends on it, changing or removing it is a breaking change, regardless of whether the original author ever meant for it to be used that way. The rule is simple to apply and easy to get wrong by default: start every class, method, and field at the most restrictive visibility that still compiles (`private`, then package-private, then `protected`, only reaching `public` when an outside caller genuinely needs it), rather than defaulting to `public` and narrowing later. A `public` class exposing `public` fields is the sharpest version of this mistake — it hands callers direct access to mutable internal state with no seam left to add validation, computed derivation, or a different representation later.

## Bad Example
```java
public class OrderProcessor {
    public List<Order> pendingOrders = new ArrayList<>(); // any caller can mutate this directly
    public OrderValidator validator = new OrderValidator(); // implementation detail, leaked

    public void validateAndQueue(Order order) {
        if (validator.isValid(order)) {
            pendingOrders.add(order);
        }
    }

    public boolean isValid(Order order) { // helper never meant to be called externally
        return validator.isValid(order);
    }
}
```

## Good Example
```java
public class OrderProcessor {
    private final List<Order> pendingOrders = new ArrayList<>();
    private final OrderValidator validator;

    public OrderProcessor(OrderValidator validator) {
        this.validator = validator;
    }

    public void validateAndQueue(Order order) {
        if (validator.isValid(order)) {
            pendingOrders.add(order);
        }
    }

    public List<Order> pendingOrders() {
        return List.copyOf(pendingOrders); // defensive copy, not the mutable internal list
    }
}
```

## Notes
- A `public` mutable field (or a getter returning a mutable internal collection directly) both hand out uncontrolled write access — expose an immutable copy or view instead (see `immutability-*` rules).
- `protected` is nearly as exposed as `public` in a widely-subclassed codebase — reserve it for members a designed-for-extension class genuinely intends subclasses to use.
- Widening visibility later is a compatible change; narrowing it is not — starting minimal costs nothing and keeps that option open.

## References
- [Effective Java, 3rd Edition — Item 15: Minimize the accessibility of classes and members](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
