---
title: Model Closed Variant Sets as a Sealed Interface, Not an Enum-and-Field Hack
impact: HIGH
impactDescription: each variant carries its own typed data instead of one enum with unused nullable fields
tags: [sealed-classes, api-design, type-safety, java17]
---

# Model Closed Variant Sets as a Sealed Interface, Not an Enum-and-Field Hack [HIGH]

## Description
An `enum` models a closed set of *values*; it cannot give different variants different typed payloads without every constant carrying every field, most of them unused and nullable for any given constant. A `sealed interface` (Java 17+) models a closed set of *types* — each `permits` clause implementation is its own record or class with exactly the fields it needs, while the interface still enumerates a fixed, compiler-known set of possibilities. Reach for a sealed interface once variants stop being interchangeable values and start being different shapes of data — a payment method with different fields per type, an event hierarchy, a parse result. Keep a plain `enum` for what it does well: an actual closed set of constants with no per-variant data.

## Bad Example
```java
public enum PaymentMethod {
    CARD, BANK_TRANSFER, PAYPAL
}

public class Payment {
    private final PaymentMethod method;
    private final String cardNumber;      // only meaningful for CARD
    private final String iban;            // only meaningful for BANK_TRANSFER
    private final String paypalEmail;     // only meaningful for PAYPAL

    // Every constructor caller must pass nulls for two of three fields,
    // and nothing stops constructing CARD with an iban instead of a cardNumber.
}
```

## Good Example
```java
public sealed interface PaymentMethod
    permits PaymentMethod.Card, PaymentMethod.BankTransfer, PaymentMethod.PayPal {

    record Card(String cardNumber, YearMonth expiry) implements PaymentMethod {}
    record BankTransfer(String iban) implements PaymentMethod {}
    record PayPal(String email) implements PaymentMethod {}
}

// Each variant is its own type: no unused fields, no invalid combinations possible
PaymentMethod method = new PaymentMethod.Card("4111111111111111", YearMonth.of(2027, 8));
```

## Notes
- `sealed` + `permits` closes the hierarchy at compile time — `switch` over it can be exhaustive with no `default` (see `sealed-pattern-switch-exhaustiveness`).
- Nesting the permitted implementations as members of the sealed interface itself (as above) is a common style choice, not a requirement — they can also be top-level types in the same file or module.
- Do not add a sealed interface for a set of types that is expected to grow from outside the module — sealing is a deliberate closed-world commitment, not a default.

## References
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
- [Effective Java, 3rd Edition — Item 34: Use enums instead of int constants](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
