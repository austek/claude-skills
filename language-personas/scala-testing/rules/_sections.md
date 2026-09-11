# Section Definitions

## Async Testing (async-testing)
**Impact:** HIGH

Test asynchronous and time-dependent code deterministically, without sleeping the test thread or reaching past the effect runner.

**Rules:**
- `async-testing-avoid-thread-sleep` - Never use Thread.sleep to wait for async work
- `async-testing-deterministic-clocks` - Test time-dependent code with a simulated clock
- `async-testing-future-recovertosucceededif` - Assert expected Future failures with recoverToSucceededIf
- `async-testing-io-testrunner` - Let CatsEffectSuite run IO tests instead of calling unsafeRunSync yourself

## Mocking Discipline (mocking-discipline)
**Impact:** HIGH

Reserve mocks for genuine external boundaries, and prefer fakes and in-memory interpreters for everything else.

**Rules:**
- `mocking-discipline-avoid-mockito-for-pure-fp` - Avoid Mockito on pure, effect-typed code
- `mocking-discipline-in-memory-interpreters` - Write in-memory interpreters for repository and client traits
- `mocking-discipline-mock-external-boundaries-only` - Reserve mocks for the external system boundary
- `mocking-discipline-prefer-fakes-over-mocks` - Prefer fakes over mocks

## ScalaCheck (scalacheck)
**Impact:** MEDIUM

Generate test inputs with ScalaCheck instead of hand-picking examples, and keep generators composable and shrinking-friendly.

**Rules:**
- `scalacheck-arbitrary-instances-for-domain-types` - Provide Arbitrary instances for domain types
- `scalacheck-forall-over-manual-loops` - Use forAll instead of hand-rolled input loops
- `scalacheck-generator-composition` - Compose generators from smaller generators
- `scalacheck-property-based-for-pure-functions` - Reach for property-based tests on pure functions
- `scalacheck-shrinking-friendly-generators` - Keep generators shrinking-friendly

## ScalaTest & MUnit Structure (scalatest-munit-structure)
**Impact:** MEDIUM

Structure ScalaTest and MUnit suites for clarity: one behavior per test, arrange-act-assert bodies, and readable naming.

**Rules:**
- `scalatest-munit-structure-aaa-pattern` - Structure test bodies as arrange-act-assert
- `scalatest-munit-structure-nested-suites` - Nest suites by shared context, not by file convenience
- `scalatest-munit-structure-one-behavior-per-test` - Assert one behavior per test
- `scalatest-munit-structure-suite-naming` - Name test suites after the unit under test
- `scalatest-munit-structure-tagged-tests` - Tag slow or environment-dependent tests

## Fixtures (fixtures)
**Impact:** MEDIUM

Scope and share test setup without leaking mutable state across tests.

**Rules:**
- `fixtures-factory-methods-for-test-data` - Build test data with factory methods, not copy-pasted literals
- `fixtures-munit-lifecycle` - Prefer FunFixture over mutable beforeEach/afterEach in MUnit
- `fixtures-narrowest-scope` - Scope fixtures to the narrowest level that needs them
- `fixtures-scalatest-beforeandafter-traits` - Match ScalaTest fixture traits to setup cost
