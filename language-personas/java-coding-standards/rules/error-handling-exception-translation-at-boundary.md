---
title: Translate Low-Level Exceptions at Architectural Boundaries
impact: HIGH
impactDescription: keeps infrastructure failure details from leaking into and coupling higher layers
tags: [error-handling, exceptions, architecture, api-design]
---

# Translate Low-Level Exceptions at Architectural Boundaries [HIGH]

## Description
When an exception crosses from one architectural layer into another — persistence into service, an HTTP client into a domain module, a third-party SDK into application code — it should be caught and translated into an exception type that belongs to the layer it is entering, not allowed to propagate in its original, implementation-specific form. Letting a `SQLException` or an SDK-specific `ApiException` leak into service or domain code couples that code to a persistence or transport detail it should not need to know about, and means changing the database driver or the HTTP client later requires hunting down every `catch` clause across the codebase that names the old exception type. Catch at the boundary, wrap in a domain-meaningful exception (per `error-handling-custom-exception-hierarchy`) that preserves the original as `cause`, and let only that translated type cross into the next layer.

## Bad Example
```java
public class OrderRepository {
    public Order findById(Long id) throws SQLException { // leaks a JDBC detail into the service layer
        try (var stmt = connection.prepareStatement("SELECT * FROM orders WHERE id = ?")) {
            stmt.setLong(1, id);
            var rs = stmt.executeQuery();
            return mapRow(rs);
        }
    }
}

public class OrderService {
    public Order getOrder(Long id) throws SQLException { // service layer now depends on JDBC
        return orderRepository.findById(id);
    }
}
```

## Good Example
```java
public class OrderRepository {
    public Order findById(Long id) {
        try (var stmt = connection.prepareStatement("SELECT * FROM orders WHERE id = ?")) {
            stmt.setLong(1, id);
            var rs = stmt.executeQuery();
            return mapRow(rs);
        } catch (SQLException e) {
            // translated at the boundary; original cause preserved for diagnostics
            throw new OrderPersistenceException("failed to load order " + id, e);
        }
    }
}

public class OrderService {
    public Order getOrder(Long id) {
        return orderRepository.findById(id); // depends only on the domain-level exception, not JDBC
    }
}
```

## Notes
- Always pass the original exception as `cause` to the wrapping exception's constructor — never drop it, or the translated exception loses the stack trace that actually pinpoints the failure.
- This applies symmetrically outbound too: a service calling a third-party HTTP SDK should translate that SDK's exceptions into the service's own exception type before they reach its own callers.
- The boundary is the right place to decide checked vs. unchecked for the translated type as well (see `error-handling-checked-vs-unchecked`) — the low-level exception's checked/unchecked status does not have to dictate the translated one's.
- Do not translate indiscriminately with a catch-all `catch (Exception e)` at the boundary — catch the specific low-level exception types the underlying API actually declares, so a genuine bug in the boundary code itself isn't silently rewrapped as an infrastructure failure.

## References
- [Effective Java, 3rd Edition — Item 73: Throw exceptions appropriate to the abstraction](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
