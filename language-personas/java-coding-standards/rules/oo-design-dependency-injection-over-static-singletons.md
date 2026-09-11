---
title: Prefer Dependency Injection Over Static Singletons
impact: HIGH
impactDescription: a dependency can be swapped for a test double without touching the class that uses it
tags: [oo-design, dependency-injection, testability, coupling]
---

# Prefer Dependency Injection Over Static Singletons [HIGH]

## Description
A class that reaches for a dependency through a static accessor (`Database.getInstance()`, a static field, a static utility method wrapping global state) hard-codes exactly which implementation it uses, at compile time, with no seam for anything else to be substituted — including a test double. That forces tests exercising the class to go through the real dependency (a real database connection, a real clock, a real HTTP client) or resort to fragile static-mocking workarounds. Passing the dependency in through a constructor makes it an explicit, visible part of the class's contract: any implementation matching the required interface works, production code passes the real one, and a test passes a stub or mock with zero special tooling.

## Bad Example
```java
public class OrderService {
    public Order placeOrder(OrderRequest request) {
        Clock clock = Clock.systemDefaultZone(); // hidden dependency, fixed at call time
        Database db = Database.getInstance();    // global singleton, no substitution point
        Order order = new Order(request, clock.instant());
        db.save(order);
        return order;
    }
}
// Testing placeOrder without a real database or the real system clock is not possible
// without static mocking.
```

## Good Example
```java
public class OrderService {
    private final Clock clock;
    private final OrderRepository repository;

    public OrderService(Clock clock, OrderRepository repository) {
        this.clock = clock;
        this.repository = repository;
    }

    public Order placeOrder(OrderRequest request) {
        Order order = new Order(request, clock.instant());
        return repository.save(order);
    }
}

// Test passes a fixed clock and an in-memory repository — no static mocking needed
OrderService service = new OrderService(
    Clock.fixed(Instant.parse("2024-01-01T00:00:00Z"), ZoneOffset.UTC),
    new InMemoryOrderRepository());
```

## Notes
- Depend on the interface, not the concrete implementation, when injecting (see `api-design-return-interface-not-impl`) — the point of injection is substitutability, which a concrete-typed constructor parameter still blocks.
- A true, deliberate singleton (one instance for the JVM's lifetime, e.g. a connection pool) can still be injected rather than accessed statically — construct it once at application startup and pass the reference down.
- A framework's dependency-injection container (Spring, CDI) automates the wiring, but the underlying principle — dependencies arrive from outside, not fetched from global state — holds whether or not a container is involved.

## References
- [Dependency Injection Principles, Practices, and Patterns — Mark Seemann](https://www.manning.com/books/dependency-injection-principles-practices-patterns)
- [Effective Java, 3rd Edition — Item 5: Prefer dependency injection to hardwiring resources](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
