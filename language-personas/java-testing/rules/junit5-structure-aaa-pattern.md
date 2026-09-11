---
title: Structure Tests with Arrange-Act-Assert
impact: HIGH
impactDescription: a reader locates the one line that matters without re-parsing the whole method
tags: [junit5, structure, readability, aaa]
---

# Structure Tests with Arrange-Act-Assert [HIGH]

## Description
Every test method should read as three visually distinct blocks: Arrange (build the inputs and collaborators), Act (invoke the one behavior under test), Assert (check the outcome). A blank line between each block is enough signal — no comment is needed to label them, and adding `// arrange` / `// act` / `// assert` labels is noise once the shape is consistent across the suite. The value is not ceremony; it is that a test failing months from now can be diagnosed by scanning three lines instead of untangling setup, invocation, and checks interleaved throughout the method.

Interleaving the phases — asserting mid-setup, mutating state after the act, invoking the subject twice — hides which call actually produced the failure and invites a test that silently checks the wrong thing. A single Act call per test also enforces `junit5-structure-one-assertion-concept`: if the method under test is invoked more than once, the test is really covering more than one concept and should split.

The pattern holds for both classic JUnit assertions and AssertJ's fluent style (`assertj-fluent-over-junit-asserts`); it is about method shape, not the assertion library. It also composes with fixtures: when Arrange grows past a couple of lines, extract it into a builder (`fixtures-builder-for-test-data`) or a `@BeforeEach` method rather than let it dominate the test's visual weight.

## Bad Example
```java
@Test
void discountAppliesToEligibleOrder() {
    Order order = new Order(customerId, List.of(item1, item2));
    assertThat(order.items()).hasSize(2);
    order.applyDiscount(discountPolicy);
    Customer customer = customerRepository.findById(customerId).orElseThrow();
    assertThat(order.total()).isEqualByComparingTo("90.00");
    assertThat(customer.tier()).isEqualTo(Tier.GOLD);
}
```

## Good Example
```java
@Test
void discountAppliesToEligibleOrder() {
    Order order = new Order(customerId, List.of(item1, item2));

    order.applyDiscount(discountPolicy);

    assertThat(order.total()).isEqualByComparingTo("90.00");
}
```

## Notes
- One Act call per test; a second invocation of the subject is a sign the test covers two behaviors.
- Assertions on setup data (confirming the fixture itself is well-formed) belong in the fixture's own test, not scattered into the Arrange block of an unrelated test.
- Keep Arrange minimal — pull unrelated collaborators from a `@BeforeEach` (see `junit5-structure-lifecycle-annotations`) so each test's Arrange shows only what that test actually varies.

## References
- [JUnit 5 User Guide — Writing Tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests)
- [Martin Fowler — GivenWhenThen](https://martinfowler.com/bliki/GivenWhenThen.html)
