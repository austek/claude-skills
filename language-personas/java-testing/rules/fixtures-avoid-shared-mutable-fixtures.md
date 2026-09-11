---
title: Avoid Shared Mutable Fixtures
impact: HIGH
impactDescription: eliminates test failures that depend on execution order or parallel scheduling
tags: [fixtures, mutability, isolation, static]
---

# Avoid Shared Mutable Fixtures [HIGH]

## Description
A fixture shared across test methods — a `static` field, an object created once in `@BeforeAll`, a singleton pulled from a shared context — is safe only if nothing ever mutates it. The moment one test method changes a shared fixture's state (adds to a shared `List`, flips a field on a shared object, advances a shared counter), every other test that reads that fixture now depends on which tests ran before it and what they did. The suite passes or fails based on execution order, and JUnit does not guarantee method execution order by default, so the failure can appear or disappear depending on the JVM, the test runner's scheduling, or whether tests run in parallel.

This failure mode is easy to introduce by accident: a `static final List<Order> orders = new ArrayList<>()` looks immutable because the reference is `final`, but the list itself is fully mutable, and `final` only prevents reassigning the reference. The same trap applies to shared mocks reused via `@BeforeAll` instead of `@BeforeEach` — a stub configured by one test's `when()` call is still configured for the next test unless something resets it. `mockito-strict-stubbing` catches some of this via `UnnecessaryStubbingException`, but a stub that happens to still apply won't be flagged even though it's leaking.

The fix is almost always to move the fixture's creation from `@BeforeAll`/`static` to `@BeforeEach`, which — combined with the default per-method test instance lifecycle (`fixtures-test-instance-per-method-default`) — guarantees a fresh, unmutated fixture for every test. Reserve genuinely shared state for things that are immutable after construction (a fixed `Clock`, a compiled regex, read-only reference data) where no test can mutate what every other test observes.

## Bad Example
```java
class OrderServiceTest {
    static final List<Order> placedOrders = new ArrayList<>(); // shared, mutable, order-dependent

    @Test
    void placingAddsToHistory() {
        placedOrders.add(service.place(order1));
        assertThat(placedOrders).hasSize(1); // fails if another test placed an order first
    }
}
```

## Good Example
```java
class OrderServiceTest {
    private List<Order> placedOrders;

    @BeforeEach
    void setUp() {
        placedOrders = new ArrayList<>(); // fresh, isolated per test
    }

    @Test
    void placingAddsToHistory() {
        placedOrders.add(service.place(order1));
        assertThat(placedOrders).hasSize(1);
    }
}
```

## Notes
- A `static` collection or mutable field initialized in `@BeforeAll` is the most common source of this bug — grep for `static` fields that aren't `final` primitives or truly immutable values.
- Enabling parallel test execution (`junit.jupiter.execution.parallel.enabled=true`) turns a merely order-dependent bug into a flaky, timing-dependent one — a strong reason to fix shared mutable fixtures before parallelizing a suite.
- Immutable shared fixtures (a `static final Clock` fixed to a known instant, a `static final` compiled `Pattern`) are fine precisely because no test can observe another test's mutation — there isn't one.

## References
- [JUnit 5 User Guide — Parallel Execution](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parallel-execution)
