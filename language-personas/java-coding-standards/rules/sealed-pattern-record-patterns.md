---
title: Destructure Sealed Record Hierarchies with Record Patterns
impact: MEDIUM
impactDescription: replaces per-branch getter calls with one exhaustive, compiler-checked destructuring switch
tags: [sealed-classes, record-patterns, pattern-matching, java21]
---

# Destructure Sealed Record Hierarchies with Record Patterns [MEDIUM]

## Description
Record patterns (Java 21+, JEP 440) let a `switch` case both match a record's type *and* bind its components in one step — `case Card(String number, YearMonth expiry)` instead of `case Card c` followed by `c.number()` and `c.expiry()` on separate lines. Combined with a sealed interface, this gives each branch direct, named access to exactly the fields that variant carries, with the same compiler-checked exhaustiveness as a plain type switch (`sealed-pattern-switch-exhaustiveness`) extended down into the shape of the data itself — a component pattern that doesn't match the record's actual component list fails to compile, not at runtime. Nested record patterns can destructure through several levels of a hierarchy in a single case, which is where this pulls the most weight over manual accessor chains.

## Bad Example
```java
sealed interface PaymentMethod permits Card, BankTransfer {}
record Card(String number, YearMonth expiry) implements PaymentMethod {}
record BankTransfer(String iban) implements PaymentMethod {}

String describe(PaymentMethod method) {
    return switch (method) {
        case Card c -> "Card ending " + c.number().substring(c.number().length() - 4)
            + " exp " + c.expiry(); // accessor calls scattered through the branch body
        case BankTransfer b -> "IBAN " + b.iban();
    };
}
```

## Good Example
```java
String describe(PaymentMethod method) {
    return switch (method) {
        case Card(String number, YearMonth expiry) ->
            "Card ending " + number.substring(number.length() - 4) + " exp " + expiry;
        case BankTransfer(String iban) -> "IBAN " + iban;
        // Component list must match Card/BankTransfer's actual components — checked at compile time
    };
}
```

## Notes
- A component pattern can itself be a type pattern or nested record pattern, e.g. `case Order(Customer(String name), _)` — the `_` unnamed pattern (Java 21+) discards a component the branch does not need.
- Use `var` in a component position (`case Card(var number, var expiry)`) to let the compiler infer each component's declared type rather than restating it.
- A guarded pattern (`case Card c when c.expiry().isBefore(now)`) adds a boolean condition to a case without giving up exhaustiveness on the unguarded cases.

## References
- [JEP 440: Record Patterns](https://openjdk.org/jeps/440)
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441)
