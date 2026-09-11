---
title: Prefer a Fake Over a Mock for Stateful Collaborators
impact: MEDIUM
impactDescription: one fake implementation replaces dozens of per-test stub setups across the whole suite
tags: [test-doubles, fake, in-memory, mockito]
---

# Prefer a Fake Over a Mock for Stateful Collaborators [MEDIUM]

## Description
A fake is a real, working implementation of an interface, simplified for tests — an `InMemoryOrderRepository` backed by a `HashMap` instead of a real database, implementing the same `OrderRepository` interface production code uses. A mock, by contrast, is behaviorless: every method returns whatever was explicitly stubbed, and nothing else. For a stateful collaborator — anything with a save-then-retrieve contract, like a repository — a mock forces every test to stub the exact sequence of calls it expects (`when(repo.findById("o-1")).thenReturn(Optional.of(order))`), which duplicates knowledge of the collaborator's behavior into every test that uses it. A fake implements that behavior once, correctly, and every test just calls its real methods — `repository.save(order); repository.findById(order.id())` genuinely returns what was saved, the same as the real implementation would.

Fakes pay off specifically when a collaborator has real internal state and multiple tests need to exercise sequences of calls against it (save, then find, then delete) — writing out that sequence as mock stubs is both more code per test and, because each test re-specifies the same contract by hand, a place where two tests can silently assume different (and equally wrong) behaviors from the same collaborator. A mock remains the right tool for boundary calls with no meaningful state — a single request/response with an external HTTP client, a notification that fires once — where there's nothing to fake and a stub-and-verify is the natural shape.

Writing a fake is an upfront cost that a mock doesn't have, so it's worth it once a collaborator's interface is used across enough tests to amortize that cost — a repository or cache used throughout a module's test suite, not a one-off interface exercised by a single test.

## Bad Example
```java
@Test
void updatingAnOrderPersistsTheChange() {
    when(repository.findById("o-1")).thenReturn(Optional.of(existingOrder));
    // every test that touches repository re-specifies find-then-save behavior by hand
    service.updateStatus("o-1", OrderStatus.SHIPPED);
    verify(repository).save(argThat(o -> o.status() == OrderStatus.SHIPPED));
}
```

## Good Example
```java
class InMemoryOrderRepository implements OrderRepository {
    private final Map<String, Order> store = new HashMap<>();

    @Override public void save(Order order) { store.put(order.id(), order); }
    @Override public Optional<Order> findById(String id) { return Optional.ofNullable(store.get(id)); }
}

@Test
void updatingAnOrderPersistsTheChange() {
    repository.save(existingOrder);

    service.updateStatus(existingOrder.id(), OrderStatus.SHIPPED);

    assertThat(repository.findById(existingOrder.id()).orElseThrow().status())
        .isEqualTo(OrderStatus.SHIPPED);
}
```

## Notes
- A fake should stay simple enough that it's obviously correct by inspection — if it grows complex enough to need its own tests, it has become production code and belongs behind its own boundary.
- `mockito-mock-boundaries-only` still applies: a fake is not an excuse to build one for every collaborator, only for stateful ones worth the upfront cost.
- Testcontainers (`testcontainers-container-reuse`) is the real-infrastructure alternative to a fake when the test specifically needs to validate against the real technology (SQL dialect quirks, actual index behavior) rather than an in-memory approximation.

## References
- [Martin Fowler — Mocks Aren't Stubs (Test Double Types)](https://martinfowler.com/articles/mocksArentStubs.html)
