---
title: Give Parameterized Tests Readable Display Names
impact: MEDIUM
impactDescription: a failing scenario is identifiable from the test report alone, no debugger needed
tags: [junit5, parametrized, display-name, readability]
---

# Give Parameterized Tests Readable Display Names [MEDIUM]

## Description
Without a custom `name`, JUnit's default display name for each `@ParameterizedTest` invocation is `[1] arg1, arg2, ...` — an index plus the raw argument values. That's enough to distinguish invocations, but it forces whoever reads a CI failure report to cross-reference invocation `[3]` against the source to figure out which scenario failed and why. `@ParameterizedTest(name = "...")` accepts a template with placeholders — `{0}`, `{1}`, ... for individual arguments, `{index}` for the invocation number, and `{arguments}`/`{argumentsWithNames}` for the full set — so the report line itself states the scenario in words.

This matters most once a parameterized test has more than two or three invocations, or arguments whose meaning isn't obvious from the raw value alone (a `Tier` enum constant is self-explanatory; a bare `BigDecimal("0.10")` next to another is not). A well-named invocation — `"applies {0}% discount to {1} tier, expecting {2}"` — turns a CI failure notification into something actionable without opening the IDE.

This is the parameterized-test analog of `junit5-structure-display-name`'s general guidance: the test's name is part of its failure output, and a name that states the behavior under test (or, here, the specific scenario) pays for itself the first time a scenario fails in CI.

## Bad Example
```java
@ParameterizedTest
@CsvSource({"GOLD, 10, 90.00", "PLATINUM, 20, 80.00"})
void discount(Tier tier, int percent, String expected) {
    assertThat(service.applyDiscount(tier, "100.00")).isEqualByComparingTo(expected);
}
// CI failure reads: "discount(Tier, int, String)[1]" — which row failed?
```

## Good Example
```java
@ParameterizedTest(name = "{0} tier gets a {1}% discount, ending at {2}")
@CsvSource({"GOLD, 10, 90.00", "PLATINUM, 20, 80.00"})
void discount(Tier tier, int percent, String expected) {
    assertThat(service.applyDiscount(tier, "100.00")).isEqualByComparingTo(expected);
}
// CI failure reads: "GOLD tier gets a 10% discount, ending at 90.00"
```

## Notes
- `{argumentsWithNames}` includes each parameter's declared name in the output, useful as a quick default before hand-writing a template.
- Keep the template a plain description of the scenario, not a restatement of the assertion — name what varies, not what the test checks.
- The default `DisplayNameGenerator` conventions (`junit5-structure-display-name`) apply to the method itself; the `name` attribute on `@ParameterizedTest` is specifically about each invocation.

## References
- [JUnit 5 User Guide — Custom Display Names](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-display-names)
