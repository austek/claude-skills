---
title: Use Vavr Either for Expected, Recoverable Failures
impact: HIGH
impactDescription: makes recoverable failure an explicit, compiler-checked return type instead of control flow via exceptions
tags: [error-handling, vavr, either, functional]
---

# Use Vavr Either for Expected, Recoverable Failures [HIGH]

## Description
For a failure that is a normal, expected outcome of calling a method — validation failing, a business rule rejecting a request, a lookup that can legitimately not find its target for reasons the caller needs to distinguish — model it as data with Vavr's `Either<L, R>` rather than throwing an exception. `Either` puts the failure type directly in the method signature (`Either<ValidationError, Order>`), so the compiler forces every caller to handle both branches, exceptions cannot be thrown-and-forgotten past a layer that meant to catch them, and the success/failure composition reads as a pipeline (`map`, `flatMap`) instead of nested `try`/`catch`. By convention, `Left` holds the failure and `Right` holds the success value. Reserve actual exceptions (checked or unchecked, per `error-handling-checked-vs-unchecked`) for cases that are not part of ordinary control flow — truly exceptional conditions and programming errors.

## Bad Example
```java
public class OrderValidator {
    public Order validate(OrderRequest request) throws ValidationException {
        if (request.items().isEmpty()) {
            throw new ValidationException("order must have at least one item");
        }
        if (request.total().compareTo(BigDecimal.ZERO) <= 0) {
            throw new ValidationException("total must be positive");
        }
        return new Order(request);
    }
}

// Caller pays exception overhead for what is really a normal branch
try {
    Order order = validator.validate(request);
    orderRepository.save(order);
} catch (ValidationException e) {
    return ResponseEntity.badRequest().body(e.getMessage());
}
```

## Good Example
```java
public class OrderValidator {
    public Either<ValidationError, Order> validate(OrderRequest request) {
        if (request.items().isEmpty()) {
            return Either.left(new ValidationError("order must have at least one item"));
        }
        if (request.total().compareTo(BigDecimal.ZERO) <= 0) {
            return Either.left(new ValidationError("total must be positive"));
        }
        return Either.right(new Order(request));
    }
}

// Both branches are explicit, no exception unwinding involved
return validator.validate(request)
    .map(orderRepository::save)
    .fold(
        error -> ResponseEntity.badRequest().body(error.message()),
        order -> ResponseEntity.ok(order));
```

## Notes
- `Either` composes: `flatMap` chains multiple fallible steps together, short-circuiting on the first `Left` without nested conditionals — closer in spirit to a `Result`/`Try` chain than to exception handling.
- `Either.right(value)`/`Either.left(error)` are the constructors; `fold(leftFn, rightFn)` is the standard way to unwrap both branches at the boundary where a concrete result (an HTTP response, a log line) is finally needed.
- Do not mix strategies within one call chain — a method returning `Either` should not also throw checked exceptions for the same class of failure; pick one signal per failure category.
- `Either` is for expected, recoverable outcomes; it is not a replacement for `Optional` (single "value or absence", see the `nullability-*` rules) nor for unchecked exceptions signaling genuine bugs.

## References
- [Vavr — Either](https://docs.vavr.io/#_either)
- [Vavr User Guide](https://www.vavr.io/vavr-docs/)
