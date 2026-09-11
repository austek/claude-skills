---
title: Test Invariants with Property-Based Testing (proptest)
impact: HIGH
impactDescription: Surfaces edge cases (empty input, overflow, unicode) a human would never hand-pick
tags: [testing, proptest, property-based-testing, fuzzing]
---

# Test Invariants with Property-Based Testing (proptest) [HIGH]

## Description
An example-based test checks one specific input against one specific expected output — it's exactly as good as the examples you thought to write, and humans are bad at imagining the inputs most likely to break code: empty collections, maximum-length strings, integer boundaries, malformed unicode. `proptest` inverts the process: instead of picking inputs, you describe the *shape* of valid inputs (a strategy) and state a property that should hold for all of them, and the library generates hundreds of randomized cases per run looking for a counterexample. When it finds one, it doesn't just report the failing input — it automatically shrinks that input down to the smallest example that still reproduces the failure, so a 200-element vector that triggers a bug gets reported as the two-element vector that's actually responsible.

## Bad Example
```rust
// Only ever exercises the cases the author happened to think of.
#[test]
fn test_reverse_twice() {
    assert_eq!("hello".chars().rev().rev().collect::<String>(), "hello");
    assert_eq!("".chars().rev().rev().collect::<String>(), "");
}
```

## Good Example
```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn reverse_reverse_is_identity(s in ".*") {
        let once: String = s.chars().rev().collect();
        let twice: String = once.chars().rev().collect();
        prop_assert_eq!(s, twice);
    }

    #[test]
    fn sort_is_idempotent(mut v in prop::collection::vec(any::<i32>(), 0..100)) {
        v.sort();
        let sorted_once = v.clone();
        v.sort();
        prop_assert_eq!(v, sorted_once);
    }
}
```

## Notes
- A handful of property shapes cover most practical cases: roundtrip (`decode(encode(x)) == x`), idempotence (`f(f(x)) == f(x)`), commutativity, and simple invariants like "pushing an element always increases length by exactly one" — reach for these before inventing a bespoke property.
- Custom input shapes come from composing existing strategies with `.prop_map(...)` (as in building a `User` from a name-and-age tuple strategy) or by deriving `Arbitrary` for a struct via `proptest-derive` when its fields are all themselves `Arbitrary`.
- `#![proptest_config(ProptestConfig { cases: 1000, ..Default::default() })]` raises the number of generated cases per test when a property is subtle enough that the default case count risks missing the failing input.
- Property tests complement example-based tests rather than replacing them — keep a few explicit examples for documentation value and known regression cases even after adding property coverage.

## References
- [test-criterion-bench](test-criterion-bench.md)
- [test-mockall-mocking](test-mockall-mocking.md)
- [test-arrange-act-assert](test-arrange-act-assert.md)
