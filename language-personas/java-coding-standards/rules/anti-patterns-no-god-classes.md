---
title: Split God Classes Along Responsibility Boundaries
impact: HIGH
impactDescription: turns one file too large for any single reviewer to hold in their head into several independently testable units
tags: [anti-patterns, oo-design, single-responsibility]
---

# Split God Classes Along Responsibility Boundaries [HIGH]

## Description
A god class accretes unrelated responsibilities over time — validation, persistence, business rules, formatting, notification — usually because adding one more method to an existing class was easier in the moment than designing a new collaborator. The symptoms compound: a large field count means most methods use only a handful of them (a sign the class is really several classes sharing one namespace), a wide, undifferentiated public surface makes the class hard to mock or fake in a test, and a high churn rate means unrelated changes collide in the same file, increasing merge conflicts and the blast radius of every change. None of this is a single bright-line size threshold — a class can be long because it's genuinely one cohesive thing (a large but focused state machine) — the actual signal is unrelated responsibilities living in one place, not raw line count.

The fix is extracting each cohesive responsibility into its own class collaborating with the others through clear interfaces, guided by the single-responsibility principle: a class should have one reason to change.

## Bad Example
```java
public class OrderManager {
    public void validateOrder(Order order) { /* ... */ }
    public void calculateTax(Order order) { /* ... */ }
    public void saveToDatabase(Order order) { /* ... */ }
    public void sendConfirmationEmail(Order order) { /* ... */ }
    public void generateInvoicePdf(Order order) { /* ... */ }
    public void applyDiscountRules(Order order) { /* ... */ }
    // one class, six unrelated reasons to change
}
```

## Good Example
```java
public class OrderValidator { public void validate(Order order) { /* ... */ } }
public class TaxCalculator { public Money calculate(Order order) { /* ... */ } }
public class OrderRepository { public void save(Order order) { /* ... */ } }
public class OrderNotifier { public void sendConfirmation(Order order) { /* ... */ } }
public class InvoiceGenerator { public byte[] generatePdf(Order order) { /* ... */ } }

public class OrderService {
    // orchestrates focused collaborators, each independently testable
    private final OrderValidator validator;
    private final TaxCalculator taxCalculator;
    private final OrderRepository repository;

    public Order place(Order order) {
        validator.validate(order);
        Money tax = taxCalculator.calculate(order);
        return repository.save(order.withTax(tax));
    }
}
```

## Notes
- `oo-design-single-responsibility` covers the underlying principle in depth — this rule is the specific anti-pattern that results from ignoring it over time.
- A large interface with unrelated method groups exhibits the same problem one layer up — split it along the same responsibility lines, and see `oo-design-favor-interfaces-for-abstraction`.
- Extraction is safest done incrementally behind existing tests, moving one responsibility at a time rather than attempting a single large rewrite.

## References
- [Clean Code — Chapter 10: Classes](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [Refactoring, 2nd Edition — Martin Fowler, "Extract Class"](https://martinfowler.com/books/refactoring.html)
