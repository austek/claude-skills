---
title: Prefer Constructor Injection Over Reflective Field Injection
impact: HIGH
impactDescription: makes every required dependency visible in the constructor signature instead of hidden across the class body
tags: [reflection, dependency-injection, immutability]
---

# Prefer Constructor Injection Over Reflective Field Injection [HIGH]

## Description
Field injection (`@Autowired private OrderRepository repository;`) relies on the framework using reflection to set a private field after construction, bypassing the constructor entirely. That has three concrete costs: the field can't be `final`, so the class loses a compile-time guarantee it was ever initialized; the class becomes uninstantiable outside the framework's reflective machinery — a plain `new OrderService()` in a unit test produces an object with every injected field `null`, forcing tests to either stand up a DI container or reflectively poke the fields themselves; and a class's true dependency list is scattered across its field declarations instead of visible in one place, making circular dependencies and overly large constructors (a sign a class is doing too much) harder to notice.

Constructor injection fixes all three: dependencies are `final`, the class is trivially constructible with plain `new` in a test by passing fakes or mocks directly, and a constructor with eight parameters is an immediate, visible prompt to reconsider the class's responsibilities.

## Bad Example
```java
@Service
public class OrderService {
    @Autowired
    private OrderRepository repository; // not final; null until the framework reflectively sets it

    @Autowired
    private PaymentGateway paymentGateway;

    public Order place(OrderRequest request) {
        // repository and paymentGateway are only valid after framework wiring runs
        return repository.save(paymentGateway.charge(request));
    }
}
```

## Good Example
```java
@Service
public class OrderService {
    private final OrderRepository repository;
    private final PaymentGateway paymentGateway;

    public OrderService(OrderRepository repository, PaymentGateway paymentGateway) {
        this.repository = repository;
        this.paymentGateway = paymentGateway;
    }

    public Order place(OrderRequest request) {
        return repository.save(paymentGateway.charge(request));
    }
}

// trivially testable without any framework:
var service = new OrderService(fakeRepository, fakePaymentGateway);
```

## Notes
- A single constructor on a Spring bean doesn't need `@Autowired` at all as of Spring 4.3+ — the framework uses it implicitly, which removes even the annotation from the picture.
- Setter injection sits between the two: it keeps fields non-final and still allows a partially constructed object, so it's appropriate only for genuinely optional dependencies with a safe default, not for required collaborators.
- See `oo-design-dependency-injection-over-static-singletons` for the broader case against reaching for a static singleton instead of any form of injection.

## References
- [Spring Framework Reference — Constructor-based vs. setter-based DI](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [Effective Java, 3rd Edition — Item 5: Prefer dependency injection to hardwiring resources](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
