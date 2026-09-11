---
title: Rely on Mockito's Strict Stubbing
impact: HIGH
impactDescription: catches stale and copy-pasted stubs at test time instead of letting them mask silently
tags: [mockito, strict-stubbing, mockitoextension]
---

# Rely on Mockito's Strict Stubbing [HIGH]

## Description
Since Mockito 2, `MockitoExtension` (and `MockitoJUnitRunner`) run with `Strictness.STRICT_STUBS` by default. Two checks come with it. First, any stub set up with `when(...).thenReturn(...)` that no test invocation actually consumes fails the test with `UnnecessaryStubbingException` — a leftover stub from a copy-pasted test, or one that no longer matches after a refactor, is reported instead of silently sitting there. Second, if a mock is invoked with arguments that don't match any stub but do resemble one closely enough to look like a mistake, Mockito raises `PotentialStubbingProblem` rather than quietly returning the type's default value (`null`, `0`, empty `Optional`) and letting the test fail somewhere downstream with a confusing NPE.

Without strict stubbing (bare `Mockito.mock()` calls with no extension, or `Strictness.LENIENT`), unused and mismatched stubs are invisible. A test can pass while asserting nothing meaningful about the interaction it claims to cover, because the stub it depends on was never actually exercised. Strict stubbing turns that into an immediate, specific failure at the point of the mistake instead of a slow leak of test confidence.

Always wire mocks through `@ExtendWith(MockitoExtension.class)` with `@Mock` fields, not manual `Mockito.mock()` calls in `@BeforeEach`, to get this validation automatically. When a stub is genuinely shared across several tests and only some of them will consume it, either move it into the individual tests that need it, or opt one stub out explicitly with `lenient().when(...)` — a deliberate, visible exception, not a blanket downgrade of the whole test class's strictness.

## Bad Example
```java
class OrderServiceTest {
    OrderRepository repository = Mockito.mock(OrderRepository.class); // no extension: no strict checks

    @BeforeEach
    void setUp() {
        when(repository.findById("o-1")).thenReturn(Optional.of(order1));
        when(repository.findById("o-2")).thenReturn(Optional.of(order2)); // unused by most tests, never flagged
    }
}
```

## Good Example
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock OrderRepository repository;
    @InjectMocks OrderService service;

    @Test
    void loadsOrderById() {
        when(repository.findById("o-1")).thenReturn(Optional.of(order1));

        Order found = service.load("o-1");

        assertThat(found).isEqualTo(order1);
    }
}
```

## Notes
- `UnnecessaryStubbingException` is reported once per test class at the end of the run, listing every unconsumed stub across all its tests.
- `PotentialStubbingProblem` only triggers when argument matchers are involved in ambiguous ways; exact-argument stubs that are simply never called still surface via `UnnecessaryStubbingException`.
- Prefer fixing the test over reaching for `lenient()` — an unused stub is almost always a sign the test's arrange section drifted from what it actually exercises.

## References
- [Mockito — Strictness](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/quality/Strictness.html)
- [Mockito JUnit 5 Extension — Javadoc](https://javadoc.io/doc/org.mockito/mockito-junit-jupiter/latest/org/mockito/junit/jupiter/MockitoExtension.html)
