---
title: Import Givens by Type, Not With a Blanket Wildcard
impact: MEDIUM
impactDescription: keeps which instance resolves at a call site traceable to a specific import line
tags: [implicits, given, imports]
---

# Import Givens by Type, Not With a Blanket Wildcard [MEDIUM]

## Description
Scala 3 makes importing implicit evidence a deliberate, separate act from importing ordinary members: an unqualified wildcard import (`import Instances.*`) brings in every `def`/`val`/`class` in `Instances` *except* its `given` instances — those need `import Instances.given` specifically. That split exists to stop implicit evidence from riding along invisibly on an import that looks like it's only there for a handful of ordinary names, but a blanket `import Instances.given` recreates the same problem one level down: it pulls in every given `Instances` defines, silently, including ones added to `Instances` later that the importing code was never written to expect.

Scala 3 supports importing givens by the type they provide evidence for — `import Instances.{given Ordering[Int]}` (or `import Instances.{given Ordering[?]}` for any `Ordering`) — which states in the import line exactly what capability is being brought into scope, the same way a specific named import states exactly what ordinary member is being used. Prefer that form, or an explicit named import when the given has a name (`import Instances.intOrdering`), over the unqualified `import Instances.given`. The unqualified form is acceptable when a module is specifically designed as a bundle of related instances meant to be imported together — a "default instances" object — where pulling in the whole bundle *is* the intent, not an accident of a broader import.

## Bad Example
```scala
object Instances:
  given Ordering[Int] with
    def compare(x: Int, y: Int): Int = x.compareTo(y)
  given JsonEncoder[User] with
    def encode(value: User): String = s"""{"name":"${value.name}"}"""

import Instances.given // both instances enter scope; unrelated code reading this import can't tell which one it needs
```

## Good Example
```scala
object Instances:
  given Ordering[Int] with
    def compare(x: Int, y: Int): Int = x.compareTo(y)
  given JsonEncoder[User] with
    def encode(value: User): String = s"""{"name":"${value.name}"}"""

import Instances.{given Ordering[Int]} // states exactly which capability this file relies on
```

## Notes
- `import Instances.{given Ordering[?]}` imports every given whose type is some `Ordering[_]`, useful when a module provides several instances of the same type class for different type parameters.
- A named given (`given intOrdering: Ordering[Int] with ...`) can also be imported by its plain name, exactly like a `def` or `val`.
- This rule mirrors the long-standing house guidance against Scala 2 `import Foo._` wildcards for ordinary members — same rationale, applied to the given-specific import form Scala 3 introduced.
- `implicits-givens-scala3-syntax` covers the `given`/`using` declaration syntax this rule's imports are targeting.

## References
- [Scala 3 Reference — Given Imports](https://docs.scala-lang.org/scala3/reference/contextual/given-imports.html)
