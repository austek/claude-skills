---
title: Leave Test Instance Lifecycle at Per-Method Default
impact: HIGH
impactDescription: prevents order-dependent test failures caused by state leaking between test methods
tags: [junit5, lifecycle, test-instance, isolation]
---

# Leave Test Instance Lifecycle at Per-Method Default [HIGH]

## Description
JUnit 5 creates a new instance of the test class for every `@Test` method by default (`TestInstance.Lifecycle.PER_METHOD`). Instance fields are therefore reset before each test runs, which is what makes `@BeforeEach` a safe place to initialize per-test state: nothing a previous test method did to an instance field can leak into the next one, because the next one runs against a fresh object entirely. This is the mechanism that makes JUnit 5 tests safe to run in any order, or in parallel, without one test's mutation corrupting another's fixture.

`@TestInstance(Lifecycle.PER_CLASS)` switches to one shared instance for every test method in the class. It exists for specific cases — sharing expensive immutable state without `static`, or using non-static `@BeforeAll`/`@AfterAll` — but it reintroduces exactly the state-leak risk per-method instantiation was built to prevent: any instance field mutated by one test is visible to every test that runs after it, in whatever order the JUnit engine happens to choose. A test suite that passes under `PER_CLASS` only because of test execution order is fragile in a way that's invisible until someone reorders, parallelizes, or runs a single test in isolation and gets a different result.

Don't reach for `PER_CLASS` to avoid repeating `@BeforeEach` setup or to make a field `static`-free — that's a cosmetic reason to accept a correctness risk. Reach for it only when a resource genuinely must be created once and safely shared (typically alongside `fixtures-avoid-shared-mutable-fixtures`'s guidance that the shared thing itself must then be immutable or otherwise safe to share).

## Bad Example
```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class OrderServiceTest {
    private final List<Order> placedOrders = new ArrayList<>(); // shared across every test

    @Test
    void firstOrderIsPlaced() {
        placedOrders.add(service.place(order1));
        assertThat(placedOrders).hasSize(1); // passes alone, fails if run after another test
    }

    @Test
    void secondOrderIsPlaced() {
        placedOrders.add(service.place(order2));
        assertThat(placedOrders).hasSize(1); // fails if firstOrderIsPlaced ran first
    }
}
```

## Good Example
```java
class OrderServiceTest {
    private final List<Order> placedOrders = new ArrayList<>(); // fresh instance per test — always empty here

    @Test
    void firstOrderIsPlaced() {
        placedOrders.add(service.place(order1));
        assertThat(placedOrders).hasSize(1);
    }

    @Test
    void secondOrderIsPlaced() {
        placedOrders.add(service.place(order2));
        assertThat(placedOrders).hasSize(1);
    }
}
```

## Notes
- `@Nested` classes inherit the enclosing class's lifecycle unless they declare their own `@TestInstance`.
- Under `PER_METHOD`, `@BeforeAll`/`@AfterAll` must be `static` because no instance exists yet when they run; that constraint disappears under `PER_CLASS`, which is the main legitimate reason to switch.
- If a field must be shared for cost reasons, prefer a `static` field with `@BeforeAll` under the default `PER_METHOD` lifecycle over switching the whole class to `PER_CLASS`.

## References
- [JUnit 5 User Guide — Test Instance Lifecycle](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle)
