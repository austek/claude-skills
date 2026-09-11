---
title: Prefix Boolean Methods and Fields with is/has/can
impact: MEDIUM
impactDescription: makes every call site read as a yes/no question, eliminating a class of double-negative bugs
tags: [naming, readability, booleans]
---

# Prefix Boolean Methods and Fields with is/has/can [MEDIUM]

## Description
A `boolean`-returning method or field reads as a predicate at every call site, and a predicate reads best as a yes/no question: `isActive()`, `hasPermission()`, `canRetry()`. A name without the prefix (`active()`, `permission()`, `retry()`) is ambiguous between "is this true" and "do this action" or "get this value" — `retry()` in particular could plausibly be an action method that performs a retry rather than a check for whether one is allowed. The prefix disambiguates at the declaration and pays off every time the method is read in an `if` condition, which for a boolean is nearly always.

The prefix also protects against double negatives: `if (!isDisabled())` still reads awkwardly, but `if (isDisabled())` and `if (!isEnabled())` both read cleanly, so naming the positive predicate (`isEnabled` over `isNotDisabled`) keeps call sites free of negation stacking.

## Bad Example
```java
public class FeatureFlag {
    private boolean enabled;

    public boolean enabled() {
        return enabled;
    }

    public boolean expired(Instant now) {
        return now.isAfter(expiry);
    }
}

if (flag.enabled() && !flag.expired(clock.instant())) { ... }
```

## Good Example
```java
public class FeatureFlag {
    private boolean enabled;

    public boolean isEnabled() {
        return enabled;
    }

    public boolean hasExpired(Instant now) {
        return now.isAfter(expiry);
    }
}

if (flag.isEnabled() && !flag.hasExpired(clock.instant())) { ... }
```

## Notes
- `has`/`can`/`should` cover the common shades of meaning beyond plain state (`hasChildren()`, `canExecute()`, `shouldRetry()`) — pick the one that reads naturally as a question.
- Record accessors are the one place Java convention drops the prefix — a `record FeatureFlag(boolean enabled)` generates `enabled()`, not `isEnabled()`; naming the component `enabled` rather than `isEnabled` avoids a doubled-up `isEnabled()` accessor.
- Avoid negative predicate names (`isNotValid`, `isDisabled` when `isEnabled` already exists) — they force a double negative at every call site that needs the opposite case.

## References
- [Google Java Style Guide — 5.2.4 Method and variable names](https://google.github.io/styleguide/javaguide.html#s5.2.4-method-variable-names)
- [Effective Java, 3rd Edition — Item 68: Adhere to generally accepted naming conventions](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
