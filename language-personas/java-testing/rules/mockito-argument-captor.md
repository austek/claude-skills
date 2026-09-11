---
title: Use ArgumentCaptor to Inspect Call Arguments
impact: MEDIUM
impactDescription: assertions target the actual argument's content instead of a hand-built equality fixture
tags: [mockito, argumentcaptor, verification]
---

# Use ArgumentCaptor to Inspect Call Arguments [MEDIUM]

## Description
`ArgumentCaptor<T>` records the actual argument passed to a mocked method during the test, so the assertion runs against what the code under test really constructed rather than against an `eq()` matcher the test author had to predict in advance. Declare a captor with `@Captor` (alongside `@ExtendWith(MockitoExtension.class)`) or `ArgumentCaptor.forClass(...)`, pass `captor.capture()` in place of an argument matcher inside `verify(mock).method(...)`, then inspect `captor.getValue()` — or `captor.getAllValues()` when the method was invoked more than once and every call's argument matters.

Captors earn their place when building the exact expected object for an `eq()` comparison is awkward: the argument carries a generated ID, a server-assigned timestamp, or other fields the test doesn't control, so no single expected instance would `equals()` it. Capturing the real argument and asserting on the fields that do matter (via AssertJ's `extracting`, see `assertj-extracting-for-collections`) sidesteps that entirely. Reach for a captor specifically to inspect arguments to a collaborator's call — not as a general substitute for `eq()`, which remains simpler and more direct whenever the expected value is fully known ahead of time.

A captor is inherently coupled to `verify()`, so it inherits the same discipline as `mockito-verify-behavior-not-implementation`: capture and assert on arguments that are part of the contract, not on every argument a method happens to receive.

## Bad Example
```java
@Test
void savesOrderWithGeneratedId() {
    service.place(command);

    // can't predict the generated id up front, so this either over-mocks id
    // generation or gives up and skips the assertion entirely
    verify(repository).save(any(Order.class));
}
```

## Good Example
```java
@Captor ArgumentCaptor<Order> orderCaptor;

@Test
void savesOrderWithGeneratedId() {
    service.place(command);

    verify(repository).save(orderCaptor.capture());
    assertThat(orderCaptor.getValue().customerId()).isEqualTo(command.customerId());
    assertThat(orderCaptor.getValue().id()).isNotNull();
}
```

## Notes
- `getAllValues()` returns arguments in invocation order across all calls that matched the `verify()`; use it when a collaborator is called multiple times with different payloads.
- A captor field must be paired with `@ExtendWith(MockitoExtension.class)` to be initialized; without it, `@Captor` fields stay `null`.
- If most fields of the captured argument matter and the type has a sane `equals()`, a plain `eq(expected)` is simpler than capturing — reach for the captor when only some fields are known ahead of time.

## References
- [Mockito — ArgumentCaptor Javadoc](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/ArgumentCaptor.html)
