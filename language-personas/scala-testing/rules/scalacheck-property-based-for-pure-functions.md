---
title: Reach for Property-Based Tests on Pure Functions
impact: HIGH
impactDescription: one property replaces dozens of hand-picked examples and finds edge cases a human wouldn't think to write
tags: [scalacheck, property-based-testing, pure-functions]
---

# Reach for Property-Based Tests on Pure Functions [HIGH]

## Description
A pure function — same input always produces the same output, no side effects — usually obeys a general law, not just a handful of memorable input/output pairs: a sort is idempotent, an encoder's output always round-trips through its decoder, a merge is commutative. When such a law exists, state it as a ScalaCheck property and let the framework generate on the order of a hundred varied inputs per run, instead of hand-picking three or four examples that happen to occur to the author. Property tests routinely surface edge cases — empty collections, negative numbers, Unicode strings, deeply nested structures — that example-based tests silently skip because nobody thought to write them.

This isn't a replacement for example-based tests: a specific regression ("this exact input crashed in production") is still best pinned down as a literal example, since a property doesn't communicate *why* one particular case mattered. Use properties for general laws the function must satisfy, and examples for named regressions and illustrative documentation.

## Bad Example
```scala
test("sortedInsert keeps the list sorted") {
  assertEquals(sortedInsert(List(1, 3, 5), 4), List(1, 3, 4, 5))
  assertEquals(sortedInsert(Nil, 1), List(1))
  // only two hand-picked cases; duplicates, negatives, and long lists are never checked
}
```

## Good Example
```scala
import org.scalacheck.Prop.forAll

class SortedInsertSuite extends munit.ScalaCheckSuite:
  property("inserting into a sorted list keeps it sorted") {
    forAll { (xs: List[Int], x: Int) =>
      val sorted = xs.sorted
      val result = sortedInsert(sorted, x)
      result.sorted == result && result.sorted(Ordering[Int]) == result
    }
  }
```

## Notes
- A property is a specification: write it as "for any valid input, this invariant holds" rather than reverse-engineering it from the implementation under test.
- `scalacheck-arbitrary-instances-for-domain-types` and `scalacheck-generator-composition` cover producing well-distributed inputs for `forAll` beyond the built-in primitive generators.
- Keep at least one example-based test per function for the "obvious" case — it documents intent for readers who won't reconstruct it from a property.

## References
- [ScalaCheck — User Guide](https://github.com/typelevel/scalacheck/blob/main/doc/UserGuide.md)
- [MUnit ScalaCheck](https://scalameta.org/munit/docs/integrations/scalacheck.html)
