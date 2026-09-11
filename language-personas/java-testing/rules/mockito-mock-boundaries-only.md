---
title: Mock Only Architectural Boundaries
impact: HIGH
impactDescription: tests survive internal refactors instead of breaking on every implementation change
tags: [mockito, mocking, boundaries, architecture]
---

# Mock Only Architectural Boundaries [HIGH]

## Description
Mock the things a unit test cannot afford to run for real: I/O, external systems, and collaborators you don't own — an HTTP client, a repository backed by a database, a message publisher, a clock, anything slow or nondeterministic. Everything on your side of that boundary — domain objects, value objects, pure helper classes, anything you can construct in a line or two — should be built for real. A real object gives you real `equals`, real validation, and real behavior for free; a mock gives you exactly the behavior you typed in, which means the test only proves the mock does what you told it to do.

Mocking past the boundary — mocking your own service classes, domain entities, or value objects — turns the test into a description of the implementation's call graph rather than a check of its behavior. Every internal refactor (extracting a method into a collaborator, inlining one, renaming a call) then breaks tests that never should have known those calls existed. The test suite becomes a tax on refactoring instead of a safety net for it.

A useful heuristic: if you can't easily instantiate the real dependency in a unit test (it opens a socket, hits a database, reads the system clock, calls a paid API), it's a boundary — mock it. If you can instantiate it in one line with no side effects, build it for real. This keeps mocks pointed outward, at the edges of the system under test, and keeps the interior of the test exercising real code. It also composes with `mockito-avoid-mocking-value-objects`: a value object is never a boundary, no matter how deep in the call graph it sits.

## Bad Example
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock Order order;              // domain object, not a boundary
    @Mock Money total;              // value object, not a boundary
    @InjectMocks OrderService service;

    @Test
    void appliesDiscount() {
        when(order.total()).thenReturn(total);
        when(total.multiply(0.9)).thenReturn(total);
        // asserts nothing about real Money arithmetic — just that mocks return mocks
    }
}
```

## Good Example
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock PaymentGateway paymentGateway;   // real boundary: external HTTP call
    @InjectMocks OrderService service;

    @Test
    void appliesDiscountToRealOrder() {
        Order order = new Order(List.of(new LineItem("sku-1", Money.of("100.00"))));

        Order discounted = service.applyDiscount(order, Percentage.of(10));

        assertThat(discounted.total()).isEqualTo(Money.of("90.00"));
    }
}
```

## Notes
- Repositories/DAOs, network clients, message queues, and `Clock`/`Instant.now()` sources are the classic boundaries.
- If a class is hard to construct for real because its constructor reaches into I/O, that's usually a design problem (inject the I/O dependency instead) rather than a reason to mock the class itself.
- Integration tests intentionally cross fewer boundaries with real implementations (see `testcontainers-lifecycle-scope`); this rule is about unit tests.

## References
- [Mockito — Official site](https://site.mockito.org/)
- [Martin Fowler — Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
