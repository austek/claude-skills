---
title: Give Each Class a Single Responsibility
impact: HIGH
impactDescription: a change to one concern touches one class instead of rippling through an unrelated one
tags: [oo-design, single-responsibility, cohesion, maintainability]
---

# Give Each Class a Single Responsibility [HIGH]

## Description
A class with more than one reason to change — it validates input *and* persists data *and* sends notifications — forces unrelated changes to collide in the same file: a change to the notification format now risks breaking persistence logic that happens to sit in the same class, and testing validation alone means dragging along a database and a mail client as collateral dependencies. A class scoped to a single responsibility changes only when that one concern changes, is testable in isolation, and its name says exactly what it does without needing "and" or "or" to describe it. Splitting by responsibility is not splitting by technical layer for its own sake — each resulting class should still be a cohesive, meaningful unit, not an arbitrary fragment.

## Bad Example
```java
public class OrderService {
    public void placeOrder(OrderRequest request) {
        if (request.items().isEmpty()) { // validation
            throw new IllegalArgumentException("order must have items");
        }
        Order order = new Order(request); // construction
        jdbcTemplate.update("INSERT INTO orders ...", order.id()); // persistence
        emailClient.send(order.customerEmail(), "Order placed"); // notification
        // Four unrelated reasons to change this class, all entangled together
    }
}
```

## Good Example
```java
public class OrderValidator {
    public Either<ValidationError, OrderRequest> validate(OrderRequest request) { /* ... */ }
}

public class OrderRepository {
    public Order save(Order order) { /* ... */ }
}

public class OrderNotifier {
    public void notifyPlaced(Order order) { /* ... */ }
}

public class OrderService {
    private final OrderValidator validator;
    private final OrderRepository repository;
    private final OrderNotifier notifier;

    public Either<ValidationError, Order> placeOrder(OrderRequest request) {
        return validator.validate(request)
            .map(Order::new)
            .map(repository::save)
            .peek(notifier::notifyPlaced);
    }
}
```

## Notes
- "Single responsibility" means one reason to change, not one method — a cohesive class can have several methods as long as they all serve the same responsibility.
- A class that needs "and" to describe what it does in one sentence is usually a sign it has more than one responsibility.
- `OrderService` above still has a job: orchestrating the others — coordination is itself a legitimate single responsibility, distinct from doing the validation, persistence, and notification work directly.

## References
- [SOLID — Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)
- [Clean Code — Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
