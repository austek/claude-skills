---
title: Never Mock Value Objects
impact: MEDIUM
impactDescription: removes a whole class of tests that pass only because the mock was told to
tags: [mockito, value-objects, records, real-objects]
---

# Never Mock Value Objects [MEDIUM]

## Description
Value objects — records, immutable data classes, anything defined by its fields rather than an identity — should always be constructed for real in tests, never mocked. A mock of a value object has no real `equals()`, `hashCode()`, or `toString()` unless each is explicitly stubbed, and every accessor returns the mock framework's default (`null`, `0`, empty) unless stubbed too. Getting a mocked value object into a usable state takes more setup code than just building the real object, and the result is less trustworthy: the test now depends on the author having stubbed every field the code under test touches, with no compiler or runtime check that the stubbed shape matches a value the real type could ever produce.

This matters most for Java records, which are value objects by construction and nearly always trivial to instantiate directly — there's rarely a reason to mock one. The same applies to any class whose entire purpose is holding data: `Money`, `Percentage`, `CustomerId`, a DTO. These types have no meaningful boundary to cross (`mockito-mock-boundaries-only`); mocking them buys nothing and loses the real arithmetic, validation, and equality the type provides.

The tell that a value object got mocked instead of built is a wall of `when(mock.field()).thenReturn(...)` calls standing in for a constructor call — every one of those lines is test setup that exists only because the object was mocked rather than instantiated, and it grows every time the type gains a field.

## Bad Example
```java
@Mock Money price;
@Mock Percentage discount;

@Test
void appliesDiscount() {
    when(price.amount()).thenReturn(new BigDecimal("100.00"));
    when(discount.rate()).thenReturn(new BigDecimal("0.10"));
    // real Money.multiply(Percentage) never runs — nothing here tests real arithmetic
}
```

## Good Example
```java
@Test
void appliesDiscount() {
    Money price = Money.of("100.00");
    Percentage discount = Percentage.of(10);

    Money discounted = price.apply(discount);

    assertThat(discounted).isEqualTo(Money.of("90.00"));
}
```

## Notes
- If a value object is expensive or awkward to construct directly, that's a sign to add a test-data builder (`fixtures-builder-for-test-data`), not to mock the type.
- Records in particular already give you structural `equals()`/`hashCode()` for free (`records-no-mutable-fields` in `java-coding-standards`) — mocking one throws that away.
- The rare legitimate exception is a value-object-shaped interface with genuinely polymorphic, non-trivial behavior (e.g., a `Strategy`); at that point it's behavior, not a value, and mocking it is fine.

## References
- [Mockito — Official site](https://site.mockito.org/)
- [Martin Fowler — Value Object](https://martinfowler.com/bliki/ValueObject.html)
