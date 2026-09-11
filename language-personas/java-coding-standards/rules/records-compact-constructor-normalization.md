---
title: Normalize Component Values in the Compact Constructor
impact: MEDIUM
impactDescription: every consumer of the record sees one canonical form, with no repeated normalization logic
tags: [records, normalization, immutability, invariants]
---

# Normalize Component Values in the Compact Constructor [MEDIUM]

## Description
Beyond rejecting invalid input, a record's compact constructor can also rewrite the incoming arguments into a canonical form before they're assigned to fields — trimming whitespace, lower-casing an identifier, rounding a scale. Because the compact constructor's parameters are implicitly reassignable and its assignment to the actual fields happens automatically afterward, this normalization runs once, at construction, and every accessor forever after returns the already-normalized value. This removes an entire category of bugs where two logically-equal values (`"USD"` vs `" usd "`) compare unequal because normalization was left to each caller to remember, inconsistently, at the point of use.

## Bad Example
```java
public record CurrencyCode(String code) {}

// Every caller must remember to normalize before constructing —
// two records holding the "same" currency can compare unequal.
CurrencyCode a = new CurrencyCode("usd");
CurrencyCode b = new CurrencyCode("USD");
a.equals(b); // false — normalization was the caller's problem, and nobody did it consistently
```

## Good Example
```java
public record CurrencyCode(String code) {
    public CurrencyCode {
        code = code.strip().toUpperCase(Locale.ROOT); // reassigns the compact ctor's parameter
        if (code.length() != 3) {
            throw new IllegalArgumentException("currency code must be 3 letters, was " + code);
        }
    }
}

CurrencyCode a = new CurrencyCode("usd");
CurrencyCode b = new CurrencyCode(" USD ");
a.equals(b); // true — both normalized to "USD" before the field was ever set
```

## Notes
- Reassigning a compact constructor's parameter (`code = code.strip()...`) is the idiomatic way to normalize — it is the parameter, not the field, that is being reassigned, and the field gets the normalized value automatically.
- Order matters when combining with `records-canonical-constructor-validation`: normalize first, then validate the normalized form, so validation logic doesn't need to account for un-normalized input.
- Keep normalization deterministic and side-effect-free — it runs on every construction, including deserialization paths that may not expect observable side effects.

## References
- [JEP 395: Records](https://openjdk.org/jeps/395)
