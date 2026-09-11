---
title: Use @Spy Sparingly
impact: MEDIUM
impactDescription: avoids the partial-mock trap where real and stubbed behavior silently mix
tags: [mockito, spy, partial-mock]
---

# Use @Spy Sparingly [MEDIUM]

## Description
`@Spy` (or `Mockito.spy(realObject)`) wraps a real instance: every method runs its real implementation unless a specific method is stubbed. That makes a spy a partial mock, and partial mocks are inherently harder to reason about than a plain mock or the real object — a reader has to know, per call, whether it hit real code or a stub. Spies exist mainly for working with legacy code that can't be refactored immediately, where isolating one troublesome method (a network call buried inside an otherwise-testable class) is more practical than restructuring the class first.

The critical gotcha: `when(spy.method()).thenReturn(...)` calls the real method first, since `when()` has to invoke the method to register the stub — if that real method has side effects (writes to disk, throws, mutates state), they happen during setup. Use `doReturn(...).when(spy).method()`, `doThrow(...)`, or `doAnswer(...)` instead, which stub without invoking the real method first. This is the opposite of the ordering for regular mocks, and forgetting it is the most common spy-related bug.

Treat a spy as a signal, not a pattern to reach for by default: needing to stub one method on an otherwise-real object usually means that method's dependency (the part you actually want to fake) should be extracted and injected, at which point a regular mock at that new seam replaces the spy entirely. A spy that survives into the long term is worth revisiting.

## Bad Example
```java
@Spy OrderService service = new OrderService(realRepository);

@Test
void placesOrder() {
    when(service.calculateShipping(order)).thenReturn(Money.of("5.00")); // invokes the REAL method first
    // ...
}
```

## Good Example
```java
@Spy OrderService service = new OrderService(realRepository);

@Test
void placesOrder() {
    doReturn(Money.of("5.00")).when(service).calculateShipping(order); // stubs without running the real method

    Order placed = service.place(order);

    assertThat(placed.total()).isEqualByComparingTo("105.00");
}
```

## Notes
- Prefer `mockito-mock-boundaries-only`: extracting the real boundary (a `ShippingCalculator` collaborator) and mocking that directly is almost always a cleaner long-term fix than spying on the class that owns it.
- `verify()` works the same way on a spy as on a mock; only the stubbing syntax (`doReturn` vs `when`) changes.
- A spy still runs real constructors and real field initialization for the wrapped object — it is not a lighter-weight mock, just a real object with selectively overridden methods.

## References
- [Mockito — Spying on Real Objects](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#spy)
