---
title: Use @EnumSource to Cover a Whole Enum's Cases
impact: MEDIUM
impactDescription: adding an enum constant later surfaces missing test coverage instead of silently skipping it
tags: [junit5, parametrized, enumsource, exhaustiveness]
---

# Use @EnumSource to Cover a Whole Enum's Cases [MEDIUM]

## Description
`@EnumSource(MyEnum.class)` drives a `@ParameterizedTest` with every constant of the given enum type, inferred automatically from the single enum-typed parameter when the class is omitted and inferrable. This is the natural fit whenever the behavior under test genuinely branches on an enum — a `switch` over `OrderStatus`, a `Tier`-based discount table — because it makes the test exhaustive by construction: adding a new constant to the enum means the parameterized test picks it up on the next run with no edit required, and a hand-written list of `@Test` methods (one per constant) would silently miss it until someone remembers to add a case.

`names` narrows the set to specific constants, and `mode` controls how: `INCLUDE` (default, run only the listed names), `EXCLUDE` (run everything except the listed names), or `MATCH_ALL`/`MATCH_ANY` for regex-based selection. `EXCLUDE` is particularly useful for "this behavior applies to every status except `CANCELLED`" style tests, since it stays correct as new constants are added — the same exhaustiveness benefit as running the full set, scoped down deliberately rather than by an incomplete `INCLUDE` list.

Reach for `@EnumSource` specifically when every constant (or an explicitly-reasoned subset) shares the same test logic; if different constants need materially different assertions, that's a sign to combine `@EnumSource` with additional `@MethodSource`-style arguments (via `@ParameterizedTest` with multiple sources isn't supported directly — pair the enum with a lookup table instead) or fall back to `@MethodSource` altogether.

## Bad Example
```java
@Test void goldGetsTenPercent() { assertDiscount(Tier.GOLD, 10); }
@Test void platinumGetsTwentyPercent() { assertDiscount(Tier.PLATINUM, 20); }
@Test void standardGetsNoDiscount() { assertDiscount(Tier.STANDARD, 0); }
// adding Tier.DIAMOND later adds no test until someone remembers to
```

## Good Example
```java
@ParameterizedTest
@EnumSource(Tier.class)
void everyTierHasADefinedDiscountRate(Tier tier) {
    Percentage rate = discountTable.rateFor(tier);

    assertThat(rate).isNotNull(); // fails immediately for any tier the table doesn't cover
}

@ParameterizedTest
@EnumSource(mode = EnumSource.Mode.EXCLUDE, names = "CANCELLED")
void nonCancelledOrdersCanBeShipped(OrderStatus status) {
    assertThat(service.canShip(anOrder().withStatus(status).build())).isTrue();
}
```

## Notes
- Combine with a readable display name (`parametrized-readable-display-names`) — `{0}` alone already reads well for enum constants, so the default is often sufficient here.
- `mode = MATCH_ANY` with a regex `names` is useful for "every constant whose name starts with `PENDING_`" style filtering without listing them individually.
- Prefer `sealed interface` + exhaustive `switch` (`java-coding-standards`'s `sealed-pattern-switch-exhaustiveness`) in production code alongside `@EnumSource` in tests — the compiler enforces exhaustiveness in the implementation, `@EnumSource` enforces it in the test.

## References
- [JUnit 5 User Guide — @EnumSource](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources-EnumSource)
