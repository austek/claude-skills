---
title: Use a Test-Data Builder Instead of Telescoping Constructors
impact: MEDIUM
impactDescription: each test's Arrange line shows only the fields that test actually varies
tags: [fixtures, builder, test-data, readability]
---

# Use a Test-Data Builder Instead of Telescoping Constructors [MEDIUM]

## Description
A test-data builder is a fluent, mutable helper — distinct from the production object's own immutable builder, if it has one — that constructs a valid instance with sensible defaults for every field, and exposes `with*` methods to override only the fields a given test cares about. Once a domain object has more than a couple of constructor parameters, calling the constructor directly in every test forces each test to spell out every field, including the ones irrelevant to what it's checking. That's noise for the reader, and it's fragile: adding a required field to the domain type means touching every call site across the suite instead of one builder.

The builder should default every field to a valid, unremarkable value (a real ID, a positive amount, a non-empty name) so that a test which only cares about, say, discount behavior can write `anOrder().withTotal("100.00").build()` and trust every other field is a valid placeholder. This keeps each test's Arrange section proportional to what the test actually varies, which is the same readability goal `junit5-structure-aaa-pattern` sets for the whole method — the builder is what keeps Arrange from growing unboundedly as the domain type grows.

Keep the builder itself simple: default values, `with*` overrides, and a `build()` call. Don't let it accumulate test-specific convenience methods like `anInvalidOrder()` for one particular test's edge case — that logic belongs in the test itself, calling `with*` overrides explicitly, so the invalid state is visible at the call site rather than hidden behind a name.

## Bad Example
```java
@Test
void appliesDiscountToEligibleOrder() {
    Order order = new Order(
        "order-1", "customer-1", List.of(new LineItem("sku-1", Money.of("100.00"))),
        OrderStatus.PLACED, Instant.parse("2026-01-01T00:00:00Z"), null, Tier.GOLD
    ); // which of these seven fields actually matters to this test?

    Order discounted = service.applyDiscount(order);

    assertThat(discounted.total()).isEqualByComparingTo("90.00");
}
```

## Good Example
```java
@Test
void appliesDiscountToEligibleOrder() {
    Order order = anOrder().withTotal("100.00").withCustomerTier(Tier.GOLD).build();

    Order discounted = service.applyDiscount(order);

    assertThat(discounted.total()).isEqualByComparingTo("90.00");
}

class OrderTestDataBuilder {
    private String id = "order-1";
    private Money total = Money.of("50.00");
    private Tier customerTier = Tier.STANDARD;

    static OrderTestDataBuilder anOrder() { return new OrderTestDataBuilder(); }

    OrderTestDataBuilder withTotal(String amount) { this.total = Money.of(amount); return this; }
    OrderTestDataBuilder withCustomerTier(Tier tier) { this.customerTier = tier; return this; }

    Order build() { return new Order(id, total, customerTier); }
}
```

## Notes
- One builder per domain type, shared across the whole test source set, is usually enough — resist per-test-class builder variants.
- For an immutable value object with only two or three fields, a plain constructor call is often clearer than a builder; reach for the builder once field count or defaulting need grows.
- Builders compose: an `OrderTestDataBuilder` can take a `LineItemTestDataBuilder` for its items, each defaulting independently.

## References
- [Martin Fowler — TestDataBuilder (via ObjectMother comparison)](https://www.natpryce.com/articles/000714.html)
