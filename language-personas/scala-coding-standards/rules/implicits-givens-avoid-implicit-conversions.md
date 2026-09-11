---
title: Avoid Implicit Conversions; Make the Conversion Explicit
impact: HIGH
impactDescription: keeps a value's actual type visible at the call site instead of silently substituted by the compiler
tags: [implicits, conversions, type-safety]
---

# Avoid Implicit Conversions; Make the Conversion Explicit [HIGH]

## Description
An implicit conversion — Scala 3's `given Conversion[A, B]` (Scala 2's implicit single-argument `def`) — lets the compiler silently insert a call converting an `A` into a `B` wherever a `B` is expected but an `A` is given, with no call-site syntax marking that anything happened. This is exactly the kind of "invisible thing running because the compiler decided to" that makes code hard to reason about from reading it alone: a method call, an assignment, or a comparison can quietly change what value actually flows through it, and tracking down which implicit conversion fired — and where it's defined — requires compiler-level reasoning a reader shouldn't need for ordinary code. It also interacts badly with type inference and overload resolution: adding an implicit conversion into scope can change which overload a call resolves to, or make previously non-compiling code compile with a different meaning, all without a corresponding change at any call site.

Scala 3 requires an explicit opt-in (`import scala.language.implicitConversions` at the point of definition) specifically to make this cost visible, but that opt-in doesn't make the technique safe to reach for by default — it only marks that it was a deliberate choice. Prefer an explicit conversion method (`def toB: B`) or a named smart constructor the caller invokes visibly. The narrow, genuinely justified cases are things like an embedded DSL's literal syntax or a shim for a Java API's expected type, where the conversion's entire job is to be invisible by design and its scope is small and well understood — not a general tool for gluing mismatched types together silently.

## Bad Example
```scala
import scala.language.implicitConversions

case class Meters(value: Double)
case class Feet(value: Double)

given Conversion[Meters, Feet] = m => Feet(m.value * 3.28084)

def reportHeight(height: Feet): String = s"${height.value} ft"

reportHeight(Meters(2.0)) // a Meters silently becomes Feet; nothing at the call site shows a conversion happened
```

## Good Example
```scala
case class Meters(value: Double):
  def toFeet: Feet = Feet(value * 3.28084)
case class Feet(value: Double)

def reportHeight(height: Feet): String = s"${height.value} ft"

reportHeight(Meters(2.0).toFeet) // the conversion is a visible, ordinary method call
```

## Notes
- `given Conversion[A, B]` is Scala 3's replacement spelling for Scala 2's implicit conversion `def`; both need an explicit language import precisely because the technique has this cost.
- Implicit conversions can shift which overload a call resolves to, or silently change what previously ambiguous or non-compiling code now means — a risk an explicit method call never carries.
- The narrow legitimate uses (embedded DSL literal syntax, Java interop shims) keep the conversion's scope small and the invisibility deliberate; general-purpose type gluing should stay a named, visible method.
- `implicits-givens-extension-methods-over-implicit-class` covers the related but distinct case of adding a *new method* to a type, which doesn't carry the same silent-substitution risk since it's always called explicitly.

## References
- [Scala 3 Reference — Implicit Conversions](https://docs.scala-lang.org/scala3/reference/contextual/conversions.html)
