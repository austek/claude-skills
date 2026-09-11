---
title: Group Related Tests with @Nested
impact: MEDIUM
impactDescription: turns a flat method-name list into a navigable, self-documenting hierarchy
tags: [junit5, structure, nested, organization]
---

# Group Related Tests with @Nested [MEDIUM]

## Description
`@Nested` marks a non-static inner class whose test methods JUnit 5 runs as a scoped group inside the outer test class. Use it to cluster tests that share a starting state or exercise the same method/scenario — "when the cart is empty," "when the user is unauthenticated" — so the grouping structure itself documents the scenarios under test, and each group can carry its own `@BeforeEach` that layers additional setup on top of the outer class's.

The inner class must not be `static` — JUnit needs an enclosing instance to invoke it — which also means a `@Nested` class cannot declare `static` members itself unless the outer class opts into `@TestInstance(Lifecycle.PER_CLASS)`. Combine `@Nested` with `@DisplayName` (`junit5-structure-display-name`) on both the inner class and its methods; the nesting turns into a readable sentence in IDE and CI test-tree output ("Empty cart" → "throws when checkout is attempted").

Reach for `@Nested` when a test class accumulates multiple *contexts* for the same subject — not merely to shorten unrelated method names. A class with one flat scenario and a handful of tests does not need nesting; forcing it in adds indirection without adding information. Nesting depth beyond two levels usually signals the production class itself has too many responsibilities.

## Bad Example
```java
class ShoppingCartTest {
    @Test
    void emptyCartCheckoutThrows() { /* ... */ }
    @Test
    void emptyCartTotalIsZero() { /* ... */ }
    @Test
    void cartWithItemsCheckoutSucceeds() { /* ... */ }
    @Test
    void cartWithItemsTotalSumsLineItems() { /* ... */ }
}
```

## Good Example
```java
class ShoppingCartTest {
    @Nested
    @DisplayName("when the cart is empty")
    class EmptyCart {
        @Test
        void checkoutThrows() { /* ... */ }
        @Test
        void totalIsZero() { /* ... */ }
    }

    @Nested
    @DisplayName("when the cart has items")
    class CartWithItems {
        @Test
        void checkoutSucceeds() { /* ... */ }
        @Test
        void totalSumsLineItems() { /* ... */ }
    }
}
```

## Notes
- A `@Nested` class inherits `@BeforeEach`/`@AfterEach` from every enclosing class, running outer callbacks before inner ones.
- `@ExtendWith` on the outer class applies to nested classes too; a nested class can add its own extensions on top.
- Don't nest merely to namespace unrelated tests — reserve it for genuinely shared context.

## References
- [JUnit 5 User Guide — Nested Tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-nested)
