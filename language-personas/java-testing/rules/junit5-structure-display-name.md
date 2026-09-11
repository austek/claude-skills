---
title: Use @DisplayName for Human-Readable Test Names
impact: LOW
impactDescription: failure reports and IDE trees read as sentences instead of camelCase identifiers
tags: [junit5, structure, display-name, readability]
---

# Use @DisplayName for Human-Readable Test Names [LOW]

## Description
`@DisplayName("...")` overrides how JUnit 5 renders a test class or method in the IDE runner, surefire/gradle reports, and CI output, without changing the method's actual identifier. It is most valuable in two places: on `@Nested` classes, where it turns the nesting into a readable sentence (`junit5-structure-nested-test-classes`), and on parametrized tests, where the default `[1] arg` rendering is opaque (`parametrized-readable-display-names`).

`@DisplayName` is a presentation layer, not a substitute for a well-named method. The method name is still what shows up in stack traces from some tools, in `grep`, and in `--tests` filters passed to the build tool — so keep it intention-revealing on its own (see `naming-intention-revealing-names` in java-coding-standards) rather than leaning on the annotation to compensate for a lazy name like `test1`. Reserve `@DisplayName` for cases where natural English genuinely reads better than any legal Java identifier could — spaces, punctuation, domain phrasing — not as a blanket habit applied to every method regardless of whether the identifier was already clear.

A `DisplayNameGenerator` (configured via `@DisplayNameGeneration` or the `junit.jupiter.displayname.generator.default` property) can derive display names automatically, e.g. replacing underscores with spaces — useful when a team standardizes on `snake_case` method names for tests specifically, but it's a project-wide choice, not a per-test one.

## Bad Example
```java
@Test
void test1() {
    assertThat(calculator.add(2, 3)).isEqualTo(5);
}
```

## Good Example
```java
@Test
@DisplayName("adding two positive numbers returns their sum")
void addingTwoPositiveNumbersReturnsTheirSum() {
    assertThat(calculator.add(2, 3)).isEqualTo(5);
}
```

## Notes
- `@DisplayName` accepts arbitrary Unicode, including emoji and punctuation — useful for grouping symbols in CI dashboards, but don't let it become the only place a test's intent is written down.
- On a `@ParameterizedTest`, prefer the `name` attribute of the source annotation (or `@DisplayName` combined with `{0}`/`{index}` placeholders) over a generic display name that hides which parameter set failed.
- `@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)` is a low-effort class-wide alternative when every method already follows `snake_case`.

## References
- [JUnit 5 User Guide — Display Names](https://junit.org/junit5/docs/current/user-guide/#writing-tests-display-names)
