---
title: Use Lifecycle Annotations for Shared Setup and Teardown
impact: MEDIUM
impactDescription: removes duplicated Arrange code and guarantees teardown runs even when a test fails
tags: [junit5, structure, lifecycle, setup, teardown]
---

# Use Lifecycle Annotations for Shared Setup and Teardown [MEDIUM]

## Description
JUnit 5 offers four lifecycle annotations with distinct scope and timing. `@BeforeEach`/`@AfterEach` run around every test method — the right place for per-test state that must not leak between tests (a fresh mock, a new instance of the subject). `@BeforeAll`/`@AfterAll` run once for the whole class and must be `static` by default, because JUnit creates a new test instance per method (`fixtures-test-instance-per-method-default`); they suit expensive, immutable, genuinely shared resources — starting a `Testcontainers` container that all tests read from, warming a cache.

Getting the scope wrong causes two different failure modes. Putting per-test mutable state in `@BeforeAll` leaks state across tests and produces order-dependent failures that vanish when a single test is run in isolation. Putting expensive, safely-shared setup in `@BeforeEach` slows the suite for no correctness benefit. `@AfterEach`/`@AfterAll` matter as much as the `@Before*` half — a container, temp file, or open connection acquired in setup needs symmetric cleanup, and JUnit runs `@AfterEach` even when the test itself throws, which manual try/finally in each test method does not reliably replicate.

Prefer extracting genuinely reusable cross-cutting setup (a database migration, a WireMock server) into a JUnit `Extension` (`fixtures-extension-for-shared-setup`) rather than copy-pasting the same `@BeforeEach` body across test classes.

## Bad Example
```java
class OrderServiceTest {
    static OrderService service;  // shared across tests, mutated by each

    @BeforeAll
    static void init() {
        service = new OrderService(new InMemoryRepository());
    }

    @Test
    void firstTest() {
        service.place(order1);  // mutates shared state other tests then see
        assertThat(service.orders()).hasSize(1);
    }
}
```

## Good Example
```java
class OrderServiceTest {
    private OrderService service;

    @BeforeEach
    void setUp() {
        service = new OrderService(new InMemoryRepository());
    }

    @Test
    void placingAnOrderAddsItToTheOrderList() {
        service.place(order1);
        assertThat(service.orders()).hasSize(1);
    }
}
```

## Notes
- `@BeforeAll`/`@AfterAll` need not be `static` under `@TestInstance(Lifecycle.PER_CLASS)`, but that switch changes instance sharing for every test in the class — see `fixtures-test-instance-per-method-default`.
- Multiple `@BeforeEach` methods (including inherited ones from `@Nested` outer classes) all run, superclass-then-subclass; there's no guaranteed order among methods at the same level, so don't let two `@BeforeEach` methods depend on each other.
- An exception thrown in `@BeforeEach` skips the test and still runs `@AfterEach`; JUnit reports the setup failure against that test.

## References
- [JUnit 5 User Guide — Test Execution Order and Lifecycle](https://junit.org/junit5/docs/current/user-guide/#writing-tests-classes-and-methods)
