---
title: Prefer Records Over Mutable POJOs for Data Carriers
impact: HIGH
impactDescription: removes hand-written boilerplate and an entire class of mutation bugs
tags: [immutability, records, api-design]
---

# Prefer Records Over Mutable POJOs for Data Carriers [HIGH]

## Description
When a class exists only to hold a fixed set of values — a DTO, a value object, a query result row — use a `record` (Java 16+) instead of a hand-written class with private fields, a constructor, getters, setters, `equals`, `hashCode`, and `toString`. A record makes every component implicitly `final`, generates canonical accessors and `equals`/`hashCode`/`toString` from the component list, and cannot silently drift out of sync the way a hand-maintained POJO can when someone adds a field and forgets to update `equals`. Reach for a regular class only when the type has real behavior beyond holding data, needs mutable state, or must control its representation independently of its constructor arguments (e.g., an entity with an identity-based `equals`).

## Bad Example
```java
public class Money {
    private BigDecimal amount;   // mutable, needs a setter to compile with frameworks
    private String currency;

    public Money(BigDecimal amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }

    public BigDecimal getAmount() { return amount; }
    public void setAmount(BigDecimal amount) { this.amount = amount; }
    public String getCurrency() { return currency; }
    public void setCurrency(String currency) { this.currency = currency; }

    @Override
    public boolean equals(Object o) {
        // hand-written, easy to forget updating when a field is added
        if (!(o instanceof Money)) return false;
        Money other = (Money) o;
        return Objects.equals(amount, other.amount) && Objects.equals(currency, other.currency);
    }

    @Override
    public int hashCode() { return Objects.hash(amount, currency); }
}
```

## Good Example
```java
public record Money(BigDecimal amount, String currency) {
    public Money {
        Objects.requireNonNull(amount, "amount");
        Objects.requireNonNull(currency, "currency");
        if (amount.scale() > 2) {
            throw new IllegalArgumentException("amount must have at most 2 decimal places");
        }
    }

    public Money plus(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("currency mismatch: " + currency + " vs " + other.currency);
        }
        return new Money(amount.add(other.amount), currency);
    }
}

// equals, hashCode, toString, and accessors amount()/currency() are generated
Money price = new Money(new BigDecimal("19.99"), "USD");
```

## Notes
- The compact constructor (`public Money { ... }`) is the place for validation — it runs before field assignment and applies to every construction path.
- Records interoperate with `sealed interface` for closed hierarchies of data shapes, matched exhaustively with `switch` — the combination the team's default stack favors over `instanceof` chains.
- A record's accessor is `amount()`, not `getAmount()` — do not add a manual `getAmount()` alongside it; that reintroduces the duplication records exist to remove.
- Records are not the right tool for JPA `@Entity` classes (which require mutable, identity-based state for the persistence provider) or for builders under construction — use a plain class there instead.

## References
- [JEP 395: Records](https://openjdk.org/jeps/395)
- [Effective Java, 3rd Edition — Item 17: Minimize mutability](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
