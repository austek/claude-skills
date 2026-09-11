---
title: Know the Difference Between Dummy, Stub, and Mock
impact: MEDIUM
impactDescription: picking the simplest sufficient double keeps each test's setup proportional to what it verifies
tags: [test-doubles, dummy, stub, mock, terminology]
---

# Know the Difference Between Dummy, Stub, and Mock [MEDIUM]

## Description
The test-double vocabulary Gerard Meszaros formalized (and Mockito's own naming loosely follows) distinguishes doubles by what they're for, and choosing the simplest one sufficient for a given test keeps that test's setup honest about what it actually checks. A **dummy** is passed only to satisfy a parameter list — it's never actually invoked or inspected, just needed to compile the call (a `null`, or a trivial instance, passed where a parameter is required but irrelevant to the test). A **stub** returns canned answers to calls made during the test, feeding the code under test a scripted input — `when(repository.findById(id)).thenReturn(Optional.of(order))` used purely so the code under test has something to work with, with no `verify()` afterward. A **mock** is a stub whose calls are then verified — the test asserts that a specific interaction happened, via `verify(...)`.

The practical distinction is where the test's real assertion lives. If the test's assertion checks the return value or resulting state, and a `when()` stub just supplies the input needed to get there, that's a stub — verifying the stub's own call afterward is redundant, since the outcome already proves the call happened correctly (`mockito-verify-behavior-not-implementation` covers this same redundancy from the verification side). If the *fact that the call happened* is itself the thing under test — a command with no return value whose only observable effect is a call to a collaborator — that's a genuine mock, and `verify()` is the correct, necessary assertion.

Mixing the two sloppily — stubbing a value the test never uses, or verifying a call whose result the test already asserted via return value — is a sign the test doesn't have a clear idea of what it's actually checking. Naming the double correctly in your own head while writing the test (dummy, stub, or mock) is a fast way to catch that before it happens.

## Bad Example
```java
@Test
void placesOrder() {
    when(repository.save(any())).thenReturn(savedOrder); // stub, used only to reach the return value

    Order result = service.place(order);

    assertThat(result).isEqualTo(savedOrder);
    verify(repository).save(order); // redundant mock-style verify — the assertion above already proves this happened
}
```

## Good Example
```java
@Test
void placesOrderAndReturnsIt() {
    when(repository.save(any())).thenReturn(savedOrder); // stub only — supplies the input, no verify needed

    Order result = service.place(order);

    assertThat(result).isEqualTo(savedOrder);
}

@Test
void placingAnOrderPublishesAnEvent() {
    service.place(order); // no return value to assert on — the call itself is the contract

    verify(eventPublisher).publish(any(OrderPlacedEvent.class)); // genuine mock usage
}
```

## Notes
- A dummy in Java is often just `null` passed for a parameter the method under test never touches — safe only when the test author has confirmed the method really doesn't use it.
- Mockito doesn't distinguish these types at the API level (`mock()` creates something that can act as any of the three depending on how it's used) — the distinction is about test intent, not tooling.
- `test-doubles-fake-over-mock` covers the fourth category, a fake, for collaborators with real state a stub/mock combination would otherwise have to reimplement per test.

## References
- [Gerard Meszaros — xUnit Test Patterns, Test Double](http://xunitpatterns.com/Test%20Double.html)
- [Martin Fowler — Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
