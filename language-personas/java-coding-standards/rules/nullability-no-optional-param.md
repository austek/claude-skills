---
title: Never Accept Optional as a Method Parameter
impact: MEDIUM
impactDescription: simpler call sites, no double-wrapping of absence
tags: [nullability, optional, api-design]
---

# Never Accept Optional as a Method Parameter [MEDIUM]

## Description
`Optional<T>` was designed as a return type, not a general-purpose "maybe" wrapper. Accepting it as a parameter pushes the wrapping cost onto every caller, who must build an `Optional` just to hand it to you, and it still does not stop a caller from passing `null` for the `Optional` reference itself — you gain a second failure mode instead of removing the first. Method overloading, or a plain nullable parameter documented with `@Nullable` plus a null check, expresses "this argument is optional" more directly and cheaper.

For a truly optional argument, prefer an overloaded method, a builder (see `immutability-builder-for-complex-construction`), or splitting the call into two explicit methods.

## Bad Example
```java
public class DiscountCalculator {
    public BigDecimal computeDiscount(Order order, Optional<Coupon> coupon) {
        // Caller must wrap, and coupon itself can still be null
        return coupon
            .map(c -> applyCoupon(order, c))
            .orElseGet(() -> defaultDiscount(order));
    }
}

// Awkward call site
calculator.computeDiscount(order, Optional.of(coupon));
calculator.computeDiscount(order, Optional.empty());
```

## Good Example
```java
public class DiscountCalculator {
    public BigDecimal computeDiscount(Order order) {
        return defaultDiscount(order);
    }

    public BigDecimal computeDiscount(Order order, Coupon coupon) {
        Objects.requireNonNull(coupon, "coupon");
        return applyCoupon(order, coupon);
    }
}

// Clear call sites, no wrapping required
calculator.computeDiscount(order);
calculator.computeDiscount(order, coupon);
```

## Notes
- If the caller already holds an `Optional<Coupon>`, it unwraps it at the call site (`coupon.map(c -> calculator.computeDiscount(order, c)).orElseGet(...)`) — the cost belongs to whoever already has the `Optional`, not to every caller.
- Static analyzers (Error Prone's `OptionalUsedAsFieldOrParameterType`, IntelliJ inspections) flag this pattern; do not suppress the warning, redesign the signature.
- An exception: builder-style fluent APIs sometimes accept `Optional` internally by convention (e.g., some request-builder DSLs) — treat that as a deliberate, documented exception, not the default.

## References
- [Effective Java, 3rd Edition — Item 55: Return optionals judiciously](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Stuart Marks — Optional design goals](https://dev.java/learn/using-optional/)
