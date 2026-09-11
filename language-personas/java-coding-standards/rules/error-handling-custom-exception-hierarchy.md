---
title: Design a Purposeful Custom Exception Hierarchy
impact: MEDIUM
impactDescription: lets callers catch precisely the failure category they can handle
tags: [error-handling, exceptions, api-design]
---

# Design a Purposeful Custom Exception Hierarchy [MEDIUM]

## Description
A module's exceptions should form a hierarchy that mirrors the distinctions callers actually need to act on differently — not a single catch-all `AppException` that tells a caller nothing beyond "something failed here", and not a flat pile of unrelated exception classes with no common root that forces callers to catch each one individually even when they'd handle them the same way. Design outward from the caller's decision points: which failures does calling code need to distinguish to choose a different response? Give each distinguishable category its own type under a shared abstract base specific to the module, and let siblings that the caller treats identically share that base without needing their own separate catch clauses.

## Bad Example
```java
// A single catch-all tells the caller nothing about what actually failed
public class OrderException extends RuntimeException {
    public OrderException(String message) { super(message); }
}

// Caller cannot distinguish "retry later" from "reject permanently" from either type or message parsing
try {
    orderService.place(request);
} catch (OrderException e) {
    log.error(e.getMessage()); // now what? every failure looks the same
}
```

## Good Example
```java
public abstract sealed class OrderException extends RuntimeException
        permits InsufficientStockException, PaymentDeclinedException, InvalidOrderException {
    protected OrderException(String message) { super(message); }
}

public final class InsufficientStockException extends OrderException {
    public InsufficientStockException(String sku) {
        super("insufficient stock for SKU " + sku);
    }
}

public final class PaymentDeclinedException extends OrderException {
    public PaymentDeclinedException(String reason) {
        super("payment declined: " + reason);
    }
}

public final class InvalidOrderException extends OrderException {
    public InvalidOrderException(String reason) {
        super("invalid order: " + reason);
    }
}

// Caller distinguishes exactly the categories it needs to act on differently
try {
    orderService.place(request);
} catch (InsufficientStockException e) {
    restockNotifier.notify(e);
} catch (PaymentDeclinedException e) {
    return ResponseEntity.status(402).body(e.getMessage());
} catch (OrderException e) {
    log.error("Order placement failed", e); // any other category in the hierarchy
}
```

## Notes
- A `sealed` exception hierarchy (Java 17+) lets a `switch` over the exception type be checked exhaustively at compile time in code paths that inspect the exception rather than catching it — consistent with the team's preference for `sealed interface` + `switch` over `instanceof` chains.
- Keep the hierarchy shallow — two levels (abstract base, concrete leaves) covers nearly every real case; deep hierarchies mostly just add navigation cost without adding caller-relevant distinctions.
- Every custom exception should support the standard `(String message, Throwable cause)` constructor pair so it can wrap an underlying cause without losing the original stack trace — see `error-handling-exception-translation-at-boundary`.
- Don't create a new exception type for a distinction no caller will ever act on differently — that is hierarchy for its own sake, not for callers' decision-making.

## References
- [Effective Java, 3rd Edition — Item 72: Favor the use of standard exceptions](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
