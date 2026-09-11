---
title: Never Let a Mutable Builder Escape Its Construction Scope
impact: HIGH
impactDescription: keeps a construction-time optimization from becoming permanent, shared mutable state
tags: [immutability, encapsulation, mutable-collections]
---

# Never Let a Mutable Builder Escape Its Construction Scope [HIGH]

## Description
Using a mutable accumulator — `ArrayBuffer`, `mutable.Map`, `StringBuilder`, a collection's own `.newBuilder` — inside a function to construct a result efficiently is a legitimate, common technique; Scala's own immutable collection implementations are built this way internally. The rule that matters is the boundary: the mutable value must never be the thing a function returns, assigns to a field, or hands to another function. Once construction is finished, convert it to its immutable counterpart (`.toList`, `.toMap`, `.result()`, `.toString()`) and return that. If the mutable builder itself escapes, every holder of the returned reference gains the ability to mutate what looks like a finished value — silently invalidating equality comparisons made earlier, breaking any caching keyed on that value, and reintroducing exactly the shared-mutation bugs immutability is meant to prevent.

This failure mode is easy to miss because it doesn't look like exposing a `var` field — the type signature can even claim to return a read-only interface (`Seq[T]`) while the concrete runtime value is still a mutable `ArrayBuffer[T]`, since Scala's mutable collections are subtypes of the corresponding immutable-looking read interfaces. A caller holding that `Seq[T]` cannot mutate it through that interface, but any other reference to the original `ArrayBuffer` — including one the constructing code kept for itself — still can, and a cast or pattern match can recover the mutable type. Treat "the concrete instance is immutable" as the actual requirement, not "the declared return type looks immutable."

## Bad Example
```scala
import scala.collection.mutable

class ReportBuilder:
  private val rowsBuffer = mutable.ArrayBuffer.empty[String]
  def addRow(row: String): Unit = rowsBuffer += row
  def rows: Seq[String] = rowsBuffer // leaks the live mutable buffer; type signature hides it
```

## Good Example
```scala
import scala.collection.mutable

class ReportBuilder:
  private val rows = mutable.ArrayBuffer.empty[String]
  def addRow(row: String): Unit = rows += row
  def build: List[String] = rows.toList // finalized, independent, genuinely immutable
```

## Notes
- A `StringBuilder` used inside a method for string concatenation is fine as long as `.toString()` is called before the method returns — never return the `StringBuilder` itself.
- Watch for the same leak through constructor parameters: accepting a `mutable.Map` and storing the reference directly (instead of copying it into an immutable `Map`) gives the caller a live handle into the object's internal state.
- `immutability-immutable-collections-default` covers the general preference for immutable collections at API boundaries; this rule is the specific trap of a temporary construction-time mutable value outliving its intended scope.
- Tools like Wartremover's `MutableDataStructures` lint flag mutable collection types in public signatures, catching the declared-type half of this problem automatically — the concrete-instance half still needs review.

## References
- [Scala Collections — Mutable and Immutable Collections](https://docs.scala-lang.org/overviews/collections-2.13/overview.html)
