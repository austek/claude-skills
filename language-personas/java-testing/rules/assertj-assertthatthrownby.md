---
title: Assert Exceptions with assertThatThrownBy
impact: HIGH
impactDescription: one fluent chain checks type, message, and cause without a separate captured-exception variable
tags: [assertj, exceptions, assertthatthrownby, error-handling]
---

# Assert Exceptions with assertThatThrownBy [HIGH]

## Description
`assertThatThrownBy(() -> subject.method())` runs the lambda, captures any thrown exception, and returns a fluent `Throwable` assertion — `isInstanceOf`, `hasMessage`/`hasMessageContaining`, `hasCauseInstanceOf`, `hasNoCause` — all chainable in one statement. It replaces the older pattern of wrapping a call in `try { ...; fail("expected exception"); } catch (Exception e) { assertEquals(...) }`, which needs a manual `fail()` call to guard against the exception not being thrown at all, and separates the "what should throw" line from the "what was thrown" checks.

JUnit's own `assertThrows(Type.class, () -> ...)` returns the caught exception for further inspection and is a reasonable alternative when only the exception type matters. `assertThatThrownBy` is preferable once the test also needs to check the message or cause, since those become chained calls on the same fluent assertion instead of separate statements against the object `assertThrows` returned. Prefer the type-narrowed `assertThatExceptionOfType(Type.class).isThrownBy(() -> ...)` form when the test's primary point is the exception type, since it puts the type first and reads slightly more declaratively — both forms are AssertJ and equally valid.

Scope the lambda passed to `assertThatThrownBy` as tightly as possible — just the one call expected to throw, not a whole block of Arrange-and-Act code — so a failure clearly means "the expected call didn't throw the expected exception," not "something earlier in the lambda threw unexpectedly."

## Bad Example
```java
@Test
void rejectsNegativeAmount() {
    try {
        Money.of("-10.00");
        fail("expected exception");
    } catch (IllegalArgumentException e) {
        assertEquals("amount must not be negative", e.getMessage());
    }
}
```

## Good Example
```java
@Test
void rejectsNegativeAmount() {
    assertThatThrownBy(() -> Money.of("-10.00"))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("amount must not be negative");
}
```

## Notes
- `hasMessageContaining` is more resilient than `hasMessage` when only part of the message is stable (e.g., it embeds a dynamic value); prefer the exact match when the whole message is a fixed contract.
- `assertThatCode(() -> ...).doesNotThrowAnyException()` is the AssertJ counterpart for asserting a call succeeds without throwing.
- The lambda runs exactly once; don't reuse the same `assertThatThrownBy` call to check multiple independent invocations — one exception assertion per Act, consistent with `junit5-structure-aaa-pattern`.

## References
- [AssertJ — Exception Assertions](https://assertj.github.io/doc/#assertj-core-exception-assertions)
