---
title: Favor Interfaces Over Abstract Classes for Abstraction
impact: MEDIUM
impactDescription: an unrelated existing class can still implement the abstraction without inheritance
tags: [oo-design, interfaces, abstraction, api-design]
---

# Favor Interfaces Over Abstract Classes for Abstraction [MEDIUM]

## Description
An abstract class forces every implementation into a single-inheritance slot it may already need for something else, and requires retrofitting an existing class into that specific class hierarchy to adopt it. An interface has neither constraint: a class can implement any number of interfaces alongside whatever it already extends, and an existing, unrelated class can adopt a new interface after the fact with no change to its inheritance chain. Java's interfaces also support `default` and `static` methods, closing most of the gap that used to justify reaching for an abstract class purely to share method implementations. Reach for an abstract class only when subclasses genuinely need to share private state or non-public helper methods across a family that is deliberately, permanently related by inheritance — not merely to define a contract.

## Bad Example
```java
public abstract class Validator {
    protected abstract boolean check(String value);

    public final Either<String, String> validate(String value) {
        return check(value) ? Either.right(value) : Either.left("invalid: " + value);
    }
}

// EmailValidator now must extend Validator — it cannot also extend
// some other class it might need, e.g. an existing FieldFormatter.
public class EmailValidator extends Validator {
    protected boolean check(String value) { return value.contains("@"); }
}
```

## Good Example
```java
public interface Validator {
    boolean check(String value);

    default Either<String, String> validate(String value) {
        return check(value) ? Either.right(value) : Either.left("invalid: " + value);
    }
}

// EmailValidator is free to extend anything else it needs, and still implements Validator
public class EmailValidator extends FieldFormatter implements Validator {
    public boolean check(String value) { return value.contains("@"); }
}
```

## Notes
- An interface cannot declare instance fields (only `static final` constants) — an abstract class is still the right tool when subclasses must share actual mutable or protected state, not just method contracts.
- `default` methods let an interface evolve with new methods without breaking existing implementers, provided a sensible default behavior exists.
- Sealed interfaces (`sealed-pattern-*` rules) combine this same abstraction benefit with a closed, exhaustively checkable set of implementations — reach for that combination when the implementer set should be fixed.

## References
- [Effective Java, 3rd Edition — Item 20: Prefer interfaces to abstract classes](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
