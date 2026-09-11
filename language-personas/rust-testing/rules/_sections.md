# Section Definitions

## Testing (test)
**Impact:** MEDIUM

Write trustworthy Rust tests with idiomatic structure, mocking, and property-based coverage.
Covers Arrange-Act-Assert structure, RAII fixtures, mockall/loom/proptest, snapshot testing, and Criterion benchmarks.

**Rules:**
- `test-arrange-act-assert` - Structure tests with arrange, act, assert
- `test-cfg-test-module` - Keep unit tests in a #[cfg(test)] module
- `test-criterion-bench` - Benchmark with Criterion, not manual timing
- `test-descriptive-names` - Name tests by the behavior they verify
- `test-doctest-examples` - Keep doc examples as executable doctests
- `test-fixture-raii` - Clean up test fixtures with RAII, not manual teardown
- `test-integration-dir` - Put integration tests in the tests/ directory
- `test-loom-concurrency` - Model-Check concurrent code with Loom
- `test-mock-traits` - Depend on traits, not concrete types, to enable mocking
- `test-mockall-mocking` - Use Mockall to generate trait mocks
- `test-proptest-properties` - Test invariants with property-based testing (proptest)
- `test-should-panic` - Assert expected panics with #[should_panic]
- `test-snapshot-testing` - Snapshot-Test large or structured output with Insta
- `test-tokio-async` - Drive async tests with #[tokio::test]
- `test-use-super` - Import parent items in tests with use super::*
