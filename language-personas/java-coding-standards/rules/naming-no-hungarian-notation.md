---
title: Do Not Encode Type or Scope in Names
impact: MEDIUM
impactDescription: removes a class of stale, misleading identifiers the compiler can't catch
tags: [naming, readability, conventions]
---

# Do Not Encode Type or Scope in Names [MEDIUM]

## Description
Hungarian notation — prefixing or suffixing a name with its type or scope (`strName`, `iCount`, `m_balance`, `lstCustomers`) — was a workaround for editors and compilers too weak to show a variable's declared type on demand. Java's type system, IDE inlay hints, and static typing make the encoding redundant the moment it's written, and worse than redundant the moment the type changes: renaming `int iCount` to a `long` leaves a lying `i` prefix that nothing forces anyone to fix, so the name actively misleads instead of staying neutral. The same argument applies to scope prefixes (`m_`, `s_`) — Java's `this.` qualifier and consistent field-access style already disambiguate a field from a local without baking scope into every identifier.

The fix is to name the thing for what it represents, and let the declared type, visibility modifier, and IDE do the job encoding used to do by hand.

## Bad Example
```java
public class strAccount {
    private double m_dblBalance;
    private List<String> lstOwners;

    public void vSetBalance(double dblAmount) {
        m_dblBalance = dblAmount;
    }
}
```

## Good Example
```java
public class Account {
    private double balance;
    private List<String> owners;

    public void setBalance(double amount) {
        this.balance = amount;
    }
}
```

## Notes
- The one encoding still worth keeping deliberately is a leading `is`/`has`/`can` on a `boolean` — that's a semantic prefix on meaning, not a type tag, and is covered separately in `naming-boolean-method-prefix`.
- Interfaces should not carry an `I` prefix (`ICustomerRepository`) — Java convention names the interface for the role (`CustomerRepository`) and, where a distinction is needed, suffixes the implementation instead (`JdbcCustomerRepository`).
- Generic type parameters keep their own short, conventional single-letter names (`T`, `E`, `K`, `V`) — that convention is about generics, not Hungarian notation, and is unrelated to this rule.

## References
- [Effective Java, 3rd Edition — Item 68: Adhere to generally accepted naming conventions](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Google Java Style Guide — 5.2 Rules by identifier type](https://google.github.io/styleguide/javaguide.html#s5.2-specific-naming-conventions)
