---
title: Use @MethodSource for Complex Parameterized Arguments
impact: MEDIUM
impactDescription: one parameterized test replaces N near-identical @Test methods
tags: [junit5, parametrized, methodsource, test-data]
---

# Use @MethodSource for Complex Parameterized Arguments [MEDIUM]

## Description
`@MethodSource` feeds a `@ParameterizedTest` from a factory method that returns a `Stream`, `Iterable`, `Collection`, or array of arguments — `Arguments.of(...)` for multiple parameters, or bare values for a single parameter. It's the right source when test inputs aren't flat primitives (`@CsvSource` territory) or a fixed enum's values (`@EnumSource`), but real objects: domain types built via a test-data builder, collections, or combinations that are easier to express in code than to encode as a string literal.

By default JUnit resolves the factory method by name within the same class; a static method reference (`@MethodSource("discountScenarios")`) is more refactor-safe than the literal string default (`"discountScenarios"` matched by name), since a rename via the string form won't be caught by the compiler. The factory method must be `static` unless the class uses `@TestInstance(Lifecycle.PER_CLASS)` (`fixtures-test-instance-per-method-default`).

`@MethodSource` is the tool for "these five scenarios are conceptually one test with different inputs" — not a place to smuggle in unrelated test cases. If the scenarios don't share the same assertion logic, they're not one parameterized test; write them as separate `@Test` methods instead. Keep the factory method itself simple — building `Arguments` instances, not exercising logic — so a reader can see every input at a glance instead of tracing through helper calls.

## Bad Example
```java
@Test void tenPercentDiscountForGoldTier() { assertDiscount(Tier.GOLD, "100.00", "90.00"); }
@Test void twentyPercentDiscountForPlatinumTier() { assertDiscount(Tier.PLATINUM, "100.00", "80.00"); }
@Test void noDiscountForStandardTier() { assertDiscount(Tier.STANDARD, "100.00", "100.00"); }
// three near-identical methods, one per tier, duplicating the same assertion
```

## Good Example
```java
@ParameterizedTest
@MethodSource("discountScenarios")
void appliesTierDiscount(Tier tier, String startingTotal, String expectedTotal) {
    Order order = anOrder().withCustomerTier(tier).withTotal(startingTotal).build();

    Order discounted = service.applyDiscount(order);

    assertThat(discounted.total()).isEqualByComparingTo(expectedTotal);
}

static Stream<Arguments> discountScenarios() {
    return Stream.of(
        Arguments.of(Tier.GOLD, "100.00", "90.00"),
        Arguments.of(Tier.PLATINUM, "100.00", "80.00"),
        Arguments.of(Tier.STANDARD, "100.00", "100.00")
    );
}
```

## Notes
- Give the parameterized test a readable display name (`parametrized-readable-display-names`) so a failing scenario is identifiable from the test report without opening the factory method.
- A factory method can be `private`; JUnit resolves it via reflection regardless of visibility, but package-private or `private` keeps it out of the class's public surface.
- For a small number of scalar arguments, `@CsvSource` (`parametrized-csv-source`) is usually more compact and just as readable — reach for `@MethodSource` once arguments stop being simple literals.

## References
- [JUnit 5 User Guide — @MethodSource](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources-MethodSource)
