---
title: Avoid getX/setX Naming for Non-JavaBean Types
impact: MEDIUM
impactDescription: names the operation instead of implying mutable, framework-managed state that may not exist
tags: [naming, immutability, records, api-design]
---

# Avoid getX/setX Naming for Non-JavaBean Types [MEDIUM]

## Description
`getX`/`setX` is a convention with a specific contract: JavaBeans, where a framework (a UI binder, a serializer, an ORM) reflectively discovers paired accessor and mutator methods on a mutable, no-arg-constructible class. Outside that contract, `getX`/`setX` naming is a false signal — it tells a reader "this is a mutable bean with a getter/setter pair" even on types that are immutable, have no setter at all, or compute the value rather than store it. A record's accessor is already named without `get` (`account.balance()`, not `account.getBalance()`) precisely because records are not beans — matching that convention across a codebase keeps immutable value types visually distinct from mutable ones.

For a plain accessor on a non-bean class, drop `get` and name the method for the value (`balance()`, `name()`); for a computed value or an action, name the method for what it does (`totalWithTax()`, not `getTotalWithTax()`) so a reader isn't misled into expecting a stored field.

## Bad Example
```java
public final class Money {
    private final BigDecimal amount;

    public Money(BigDecimal amount) {
        this.amount = amount;
    }

    public BigDecimal getAmount() { // reads as a bean getter on a class with no setter
        return amount;
    }

    public BigDecimal getAmountWithTax(BigDecimal rate) { // not a stored field at all
        return amount.multiply(BigDecimal.ONE.add(rate));
    }
}
```

## Good Example
```java
public final class Money {
    private final BigDecimal amount;

    public Money(BigDecimal amount) {
        this.amount = amount;
    }

    public BigDecimal amount() {
        return amount;
    }

    public BigDecimal amountWithTax(BigDecimal rate) {
        return amount.multiply(BigDecimal.ONE.add(rate));
    }
}
```

## Notes
- Keep `getX`/`setX` where the bean contract is genuinely required — a JPA entity, a JAXB/Jackson-mapped DTO relying on reflective bean discovery, or a framework that documents the convention as mandatory.
- Records generate this accessor style automatically; see `records-no-mutable-fields` for when a record fits in place of a hand-written immutable class.
- `is`-prefixed boolean accessors (`isActive()`) are a separate, still-conventional case — see `naming-boolean-method-prefix`.

## References
- [JavaBeans Specification 1.01](https://download.oracle.com/otndocs/jcp/7224-javabeans-1.01-fr-spec-oth-JSpec/)
- [Effective Java, 3rd Edition — Item 51: Design method signatures carefully](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
