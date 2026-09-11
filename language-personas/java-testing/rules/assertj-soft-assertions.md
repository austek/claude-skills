---
title: Use SoftAssertions to Report Every Failure in One Run
impact: MEDIUM
impactDescription: one failing test run reports every broken field instead of one at a time across N reruns
tags: [assertj, soft-assertions, multi-field]
---

# Use SoftAssertions to Report Every Failure in One Run [MEDIUM]

## Description
A normal AssertJ assertion throws immediately on the first failure, so when a test checks several independent fields of one result, only the first mismatch is ever reported — fixing it and rerunning is the only way to discover the second. `SoftAssertions` (or the `@ExtendWith(SoftAssertionsExtension.class)` + `@InjectSoftAssertions` combination, or the static `assertSoftly(softly -> { ... })` form) collects every assertion's outcome without throwing per-call, and reports all failures together at the end of the block — one test run tells the whole story instead of one failure at a time.

This is specifically for asserting several independent properties of the *same* logical outcome — multiple fields of one result object, several unrelated invariants a single Act call should establish. It is not a way to combine multiple unrelated behaviors into one test method; `junit5-structure-one-assertion-concept` still applies to what the test is conceptually checking. A soft-assertion block checking five fields of one `Order` is one concept ("the order was built correctly") expressed as five checks; a soft-assertion block spanning two different Act calls on two different subjects is two tests that happen to share a method.

`assertSoftly` is usually the most convenient form — it opens a lambda scope, runs the block, and calls `assertAll()` automatically at the end, so there's no separate cleanup step to forget the way manual `SoftAssertions` instances require.

## Bad Example
```java
@Test
void orderIsBuiltCorrectly() {
    Order order = service.build(command);

    assertThat(order.customerId()).isEqualTo("cust-1");
    assertThat(order.total()).isEqualByComparingTo("90.00"); // never reached if customerId already failed
    assertThat(order.status()).isEqualTo(OrderStatus.PLACED);
}
```

## Good Example
```java
@Test
void orderIsBuiltCorrectly() {
    Order order = service.build(command);

    assertSoftly(softly -> {
        softly.assertThat(order.customerId()).isEqualTo("cust-1");
        softly.assertThat(order.total()).isEqualByComparingTo("90.00");
        softly.assertThat(order.status()).isEqualTo(OrderStatus.PLACED);
    }); // reports every failing line in one run
}
```

## Notes
- `@ExtendWith(SoftAssertionsExtension.class)` with an `@InjectSoftAssertions SoftAssertions softly` field auto-verifies at the end of the test method without an explicit `assertSoftly` block, useful when a class uses soft assertions in most of its tests.
- Soft assertions still fail the test — they only defer reporting to the end of the block, not skip it.
- Prefer plain hard assertions (the default) for a single-value check; soft assertions add ceremony that only pays off once there's more than one independent thing to check.

## References
- [AssertJ — Soft Assertions](https://assertj.github.io/doc/#assertj-core-soft-assertions)
