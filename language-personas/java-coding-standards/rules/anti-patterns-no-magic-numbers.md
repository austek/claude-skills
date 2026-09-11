---
title: Replace Magic Numbers and Strings with Named Constants
impact: MEDIUM
impactDescription: turns a value's meaning into something grep and the compiler both understand, instead of tribal knowledge
tags: [anti-patterns, naming, readability]
---

# Replace Magic Numbers and Strings with Named Constants [MEDIUM]

## Description
A literal like `86400`, `0.07`, or `"ADMIN"` embedded directly in an expression carries no information about why that specific value was chosen or what it represents — a reader has to infer from context (or ask someone) whether `86400` is a number of seconds in a day, a byte limit, or an unrelated ID, and that inference has to be redone at every call site the literal appears in. A named constant (`SECONDS_PER_DAY`, `STANDARD_TAX_RATE`, `Role.ADMIN`) answers the question once, at the declaration, and every use site inherits that meaning for free. It also collapses duplication: the same magic number copy-pasted across several methods becomes a single source of truth that a future change updates in one place instead of hunting down every occurrence, some of which are easy to miss.

This isn't a call to eliminate every literal — `0`, `1`, and `-1` in their conventional roles (an empty check, an increment, "not found") are usually clear from context and naming them often adds noise rather than removing it. The rule targets a value whose meaning isn't obvious from its immediate context, and especially one that's duplicated or that encodes a business rule likely to change.

## Bad Example
```java
public boolean isEligibleForDiscount(Customer customer) {
    return customer.accountAgeInDays() > 365 && customer.totalSpend() > 5000;
}

public Duration sessionTimeout() {
    return Duration.ofSeconds(1800);
}
```

## Good Example
```java
private static final int LOYALTY_THRESHOLD_DAYS = 365;
private static final BigDecimal LOYALTY_SPEND_THRESHOLD = BigDecimal.valueOf(5000);
private static final Duration SESSION_TIMEOUT = Duration.ofMinutes(30);

public boolean isEligibleForDiscount(Customer customer) {
    return customer.accountAgeInDays() > LOYALTY_THRESHOLD_DAYS
            && customer.totalSpend().compareTo(LOYALTY_SPEND_THRESHOLD) > 0;
}

public Duration sessionTimeout() {
    return SESSION_TIMEOUT;
}
```

## Notes
- An `enum` is usually the better fit for a fixed set of related named values (roles, statuses, categories) rather than a set of unrelated `String`/`int` constants — it also gets exhaustiveness checking in a `switch`, see `sealed-pattern-switch-exhaustiveness`.
- A constant's name should describe what the value means, not restate the value — `SECONDS_PER_DAY = 86400`, not `EIGHTY_SIX_THOUSAND_FOUR_HUNDRED = 86400`.
- Comment discipline applies here too: cite the constant in a comment, never repeat its literal value — a comment saying "must match the value in X" stays true even if the constant changes; one repeating the number goes stale immediately.

## References
- [Effective Java, 3rd Edition — Item 34: Use enums instead of int constants](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Clean Code — Chapter 17: Smells and Heuristics (G25: Replace Magic Numbers with Named Constants)](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
