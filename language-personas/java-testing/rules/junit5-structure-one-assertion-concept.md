---
title: Assert One Behavioral Concept Per Test
impact: HIGH
impactDescription: a failing test names the broken behavior directly, cutting diagnosis time
tags: [junit5, structure, assertions, single-responsibility]
---

# Assert One Behavioral Concept Per Test [HIGH]

## Description
"One assertion per test" is a misleading shorthand for the real rule: one *behavioral concept* per test. A single concept can legitimately require several `assertThat` calls — checking multiple fields of one resulting object, for instance — as long as they all verify the same outcome of the same Act. What must not happen is a test method that exercises the subject once and then asserts on several *unrelated* concerns (the discount was applied, the audit log recorded it, the customer tier changed), because when it fails, the method name and the stack trace don't say which of those three concerns broke — someone has to read the whole body to find out.

The failure mode this guards against is a test suite that "passes" as a checklist but degrades as a diagnostic tool: a red test should name the misbehaving concept before anyone opens the file. AssertJ's `SoftAssertions` (`assertj-soft-assertions`) is not an escape hatch from this rule — it changes how multiple assertions *of the same concept* report together, it does not license bundling unrelated concepts into one test for convenience.

Split by concept, not by counting `assertThat` calls: `appliesDiscountToTotal`, `recordsDiscountInAuditLog`, and `upgradesCustomerTierOnThreshold` are three tests even though they may share the same Arrange and Act, because each names and isolates a distinct thing that can independently break.

## Bad Example
```java
@Test
void applyingDiscount() {
    order.applyDiscount(discountPolicy);

    assertThat(order.total()).isEqualByComparingTo("90.00");
    assertThat(auditLog.entries()).hasSize(1);
    assertThat(customerRepository.findById(customerId).orElseThrow().tier())
        .isEqualTo(Tier.GOLD);
}
```

## Good Example
```java
@Test
void appliesDiscountToOrderTotal() {
    order.applyDiscount(discountPolicy);

    assertThat(order.total()).isEqualByComparingTo("90.00");
}

@Test
void recordsAppliedDiscountInAuditLog() {
    order.applyDiscount(discountPolicy);

    assertThat(auditLog.entries()).hasSize(1);
}
```

## Notes
- Several `assertThat` calls checking different fields of the *same* returned object are one concept, not several — don't over-split to the point of one field per test method.
- A method name that needs "and" to describe it (`appliesDiscountAndRecordsAudit`) is a strong signal the test covers two concepts and should split.
- Pairs naturally with `junit5-structure-aaa-pattern`: one Act call per test makes it structurally harder to smuggle in unrelated assertions.

## References
- [JUnit 5 User Guide — Writing Tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests)
