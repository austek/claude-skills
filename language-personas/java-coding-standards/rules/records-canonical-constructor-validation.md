---
title: Validate Invariants in the Record's Canonical Constructor
impact: HIGH
impactDescription: an invalid record instance becomes impossible to construct, not just discouraged
tags: [records, validation, invariants, immutability]
---

# Validate Invariants in the Record's Canonical Constructor [HIGH]

## Description
A record's canonical constructor is the one path every instance is built through — there is no way to obtain a `Money` or an `EmailAddress` record that bypassed it, unlike a class where a subclass or a reflection-based framework can sometimes sidestep validation logic placed elsewhere. That makes it the correct, and only necessary, place to enforce a record's invariants: null checks, range checks, format checks. Validating here means every caller gets the same guarantee for free — a record that constructs successfully is valid by construction, and code consuming it never needs to re-check what the constructor already enforced.

## Bad Example
```java
public record Percentage(int value) {}

// Nothing stops constructing an invalid Percentage — callers must remember to check
Percentage discount = new Percentage(150); // no compile or construction-time error
if (discount.value() < 0 || discount.value() > 100) {
    throw new IllegalArgumentException("invalid percentage"); // validation lives far from construction
}
```

## Good Example
```java
public record Percentage(int value) {
    public Percentage {
        if (value < 0 || value > 100) {
            throw new IllegalArgumentException("value must be 0-100, was " + value);
        }
    }
}

// Every construction path is validated — an invalid Percentage cannot exist
Percentage discount = new Percentage(150); // throws IllegalArgumentException immediately
```

## Notes
- Use the compact form (`public Percentage { ... }`, no parameter list) for validation — it runs before field assignment and applies to every construction path, including the implicit all-args one.
- `Objects.requireNonNull` for each reference-typed component belongs here too, not scattered across accessor call sites later.
- A validation failure should throw immediately from the constructor (an unchecked exception is standard for a rejected constructor argument) — a record's constructor is not the place to return `Either`, since construction itself either succeeds or it doesn't.

## References
- [JEP 395: Records](https://openjdk.org/jeps/395)
- [Effective Java, 3rd Edition — Item 50: Make defensive copies when needed](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
