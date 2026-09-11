---
title: Use @CsvSource for Compact Scalar Argument Tables
impact: MEDIUM
impactDescription: a test's whole input/output table is visible on the annotation, no factory method to open
tags: [junit5, parametrized, csvsource, test-data]
---

# Use @CsvSource for Compact Scalar Argument Tables [MEDIUM]

## Description
`@CsvSource` inlines a table of arguments directly on the test method as comma-separated string literals, one line per invocation. For scenarios whose inputs are primitives, strings, or enums — no object construction needed — it keeps the entire input/output table visible at the call site, which is more scannable than a `@MethodSource` factory method a reader has to jump to. JUnit converts each column to the target parameter type using its built-in implicit converters (`String` to `int`, `enum`, `LocalDate`, and others), so most scalar parameter types work without extra configuration.

Use `delimiter` or `delimiterString` to change the separator when a value naturally contains a comma, and a leading `#` on a line for a comment describing that row when the columns alone aren't self-explanatory. `@CsvFileSource` is the sibling annotation for the same format read from a `.csv` classpath resource — reach for it once the table grows large enough that embedding it in the annotation hurts readability, typically dozens of rows.

`@CsvSource` stops being the right tool once an argument needs to be a constructed object, a collection, or anything that can't be expressed as a short string literal — at that point `@MethodSource` (`parametrized-method-source`) is the correct escalation, not increasingly contorted string encoding crammed into a CSV cell.

## Bad Example
```java
@ParameterizedTest
@MethodSource("vatScenarios")
void computesVat(BigDecimal net, BigDecimal rate, BigDecimal expected) {
    assertThat(service.applyVat(net, rate)).isEqualByComparingTo(expected);
}

static Stream<Arguments> vatScenarios() {
    return Stream.of(
        Arguments.of(new BigDecimal("100.00"), new BigDecimal("0.20"), new BigDecimal("120.00")),
        Arguments.of(new BigDecimal("50.00"), new BigDecimal("0.10"), new BigDecimal("55.00"))
    ); // a factory method for what is really a flat table of numbers
}
```

## Good Example
```java
@ParameterizedTest
@CsvSource({
    "100.00, 0.20, 120.00",
    "50.00,  0.10, 55.00"
})
void computesVat(BigDecimal net, BigDecimal rate, BigDecimal expected) {
    assertThat(service.applyVat(net, rate)).isEqualByComparingTo(expected);
}
```

## Notes
- Use the two-argument form `"sku-1, "` (trailing comma, empty string) to pass an empty string, and `@CsvSource(value = {...}, nullValues = "N/A")` to pass an explicit `null` for a placeholder token.
- Whitespace around commas is trimmed by default; align columns for readability without affecting parsed values.
- `@CsvFileSource(resources = "/vat-scenarios.csv")` moves a large table to a classpath file, keeping the same column semantics.

## References
- [JUnit 5 User Guide — @CsvSource](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources-CsvSource)
