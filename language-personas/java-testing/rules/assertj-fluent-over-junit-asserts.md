---
title: Prefer AssertJ's Fluent Assertions Over JUnit's assertEquals
impact: HIGH
impactDescription: failure messages describe the actual mismatch instead of two opaque toString() dumps
tags: [assertj, fluent, readability, failure-messages]
---

# Prefer AssertJ's Fluent Assertions Over JUnit's assertEquals [HIGH]

## Description
JUnit's static `Assertions.assertEquals(expected, actual)` (and its siblings `assertTrue`, `assertNotNull`) carry two problems: the argument order is easy to get backwards with no compiler check, and a failure reports only `expected: <X> but was: <Y>` using each object's `toString()` — for a collection, a domain object without a hand-written `toString()`, or a nested structure, that message often doesn't say what actually differed. AssertJ's `assertThat(actual).isEqualTo(expected)` fixes the argument-order ambiguity by construction — there's only one subject — and its type-specific assertions (`hasSize`, `contains`, `containsExactly`, `isEqualByComparingTo`) produce failure messages that name the specific mismatch: which element was missing, which field differed, which size was expected versus found.

The fluent chain also reads closer to a sentence than a static method call, and it composes: `assertThat(order.items()).hasSize(2).extracting(LineItem::sku).contains("sku-1")` expresses several checks in one readable statement rather than several `assertEquals` calls each re-deriving the subject. For collections specifically, prefer `containsExactly`/`containsExactlyInAnyOrder` over comparing sizes and contents in separate assertions — one AssertJ call replaces what would otherwise be several JUnit assertions and reports a single, precise diff on failure.

This isn't about banning JUnit's `Assertions` entirely — `assertTimeout` has no AssertJ equivalent and remains reasonable — but for equality, collection, and object-state checks, AssertJ's fluent API is the default. Exception assertions have their own fluent AssertJ replacement, `assertThatThrownBy` (`assertj-assertthatthrownby`), which is preferable to `assertThrows` once a test also needs to check the exception's message or cause.

## Bad Example
```java
@Test
void orderTotalsCorrectly() {
    List<LineItem> items = order.items();
    assertEquals(2, items.size());
    assertTrue(items.contains(new LineItem("sku-1", Money.of("10.00"))));
    assertEquals(new BigDecimal("90.00"), order.total().amount()); // which arg is expected?
}
```

## Good Example
```java
@Test
void orderTotalsCorrectly() {
    assertThat(order.items()).hasSize(2).extracting(LineItem::sku).contains("sku-1");
    assertThat(order.total()).isEqualByComparingTo("90.00");
}
```

## Notes
- `isEqualByComparingTo` (not `isEqualTo`) is the correct choice for `BigDecimal` and other `Comparable` types where scale differences (`"90.00"` vs `"90.0"`) shouldn't fail an otherwise-equal comparison.
- `assertThatThrownBy` (`assertj-assertthatthrownby`) replaces `assertThrows` with the same fluency benefit for exception assertions.
- AssertJ assertions still throw `AssertionError` on failure, so they integrate with JUnit's reporting without any extra configuration.

## References
- [AssertJ — Core Assertions Guide](https://assertj.github.io/doc/#assertj-core)
