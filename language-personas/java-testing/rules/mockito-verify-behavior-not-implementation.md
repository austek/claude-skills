---
title: Verify Behavior, Not Implementation
impact: HIGH
impactDescription: tests keep passing across implementation refactors that preserve behavior
tags: [mockito, verify, behavior, coupling]
---

# Verify Behavior, Not Implementation [HIGH]

## Description
`verify()` exists to confirm that a side-effecting interaction the contract actually promises took place — an email got sent, a row got saved, an event got published. It is not a substitute for asserting the return value, and it should not be used to check every call a method happens to make internally. A test that verifies incidental interactions (a getter invoked while building a log message, a call whose only purpose is to compute something already checked via the return value) is asserting on the current implementation's call graph rather than on its observable behavior. The next equivalent refactor — reordering two calls, inlining a helper, fetching a value a different way — breaks the test even though nothing externally visible changed.

Prefer asserting the return value or the resulting state whenever the method produces one; reach for `verify()` specifically for commands and void methods whose only observable effect is a call to a collaborator. When verification is warranted, verify the minimum that establishes the contract: the right method, on the right mock, with the right arguments (via `eq()`, an `ArgumentCaptor`, or an argument matcher) — not `times()` counts or ordering unless the contract genuinely depends on them.

`verifyNoMoreInteractions()` and `verifyNoInteractions()` are the most over-specifying calls Mockito offers: they fail on any interaction the test didn't anticipate, including ones added later for legitimate reasons unrelated to the behavior under test (a new logging call, a new metric). Use them only when "nothing else happens" is itself the contract being tested — for example, verifying a guard clause short-circuits before touching any collaborator.

## Bad Example
```java
@Test
void placesOrder() {
    service.place(order);

    verify(repository).save(order);
    verify(order).items();          // incidental — irrelevant to the contract
    verify(order).customerId();     // incidental
    verifyNoMoreInteractions(order);
}
```

## Good Example
```java
@Test
void placesOrder() {
    service.place(order);

    verify(repository).save(order);
}

@Test
void guardRejectsOrderWithNoItems() {
    Order empty = new Order(List.of());

    assertThatThrownBy(() -> service.place(empty))
        .isInstanceOf(IllegalArgumentException.class);

    verifyNoInteractions(repository); // the contract IS "nothing gets persisted"
}
```

## Notes
- If both a return value and a side effect matter, assert the return value directly and `verify()` only the side effect — don't re-derive the return value through more `verify()` calls.
- `ArgumentCaptor` (`mockito-argument-captor`) pairs naturally with `verify()` when the argument's content matters more than its identity.
- Avoid `verify(mock, times(n))` unless the exact call count is part of the documented contract; `atLeastOnce()` or a captured-list assertion is often a better fit for "was called, with these values."

## References
- [Mockito — Verification Javadoc](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#verification)
- [Martin Fowler — Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
