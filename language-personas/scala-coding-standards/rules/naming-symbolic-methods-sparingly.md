---
title: Reserve Symbolic Method Names for Well-Established Operators
impact: MEDIUM
impactDescription: keeps a method call greppable and its meaning inferable without opening the type's definition
tags: [naming, style, operators, readability]
---

# Reserve Symbolic Method Names for Well-Established Operators [MEDIUM]

## Description
Scala allows a method name to be a symbol (`+`, `|+|`, `:+`, `<*>`), and a small set of these are effectively universal vocabulary: `+`/`-` for addition-like combination, `::`/`:+`/`++` for the collection operations they already denote in the standard library, `|+|` for a `Semigroup` combine in cats-derived code. Using one of these where it means what it conventionally means costs a reader nothing — `money1 + money2` reads exactly as expected, and looking up the operator's meaning is rarely necessary because the convention already carries it. An invented symbolic name for a domain operation, on the other hand, has no such shared meaning to lean on: `cart --> checkout` or `order >> shipment` might make sense to whoever wrote them, but a reader (or a text search across the codebase) has no way to know what the symbol does or even find its definition without already knowing which type declares it, since a symbol doesn't show up in searches the way a named method does.

The practical line: reach for a symbolic method name only when it's already conventional for the operation it performs — arithmetic-like combination, algebraic structures from a well-known library (cats' `Semigroup`/`Monoid`/`Applicative` operators), or standard-library-established collection operators reused on a custom collection type with matching semantics. For anything domain-specific — a business rule, a workflow transition, a validation step — a named method (`combine`, `andThen`, `authorize`) documents its own purpose in a way no symbol can, and stays searchable by every tool a codebase already uses to search for named things.

## Bad Example
```scala
case class Money(cents: Long):
  def -->(other: Money): Money = Money(cents + other.cents) // what does --> mean here? nothing conventional says so
```

## Good Example
```scala
case class Money(cents: Long):
  def plus(other: Money): Money = Money(cents + other.cents)
  def +(other: Money): Money = plus(other) // + is a well-established, universally understood operator for this
```

## Notes
- A symbolic method is never required to stand alone — pairing it with a named equivalent (`+`/`plus` above) keeps both the conventional operator syntax and a searchable, self-explaining name available to callers.
- This applies most sharply to public APIs; a private, tightly scoped DSL internal to one module has more room for symbolic operators its narrow audience already understands, though even then a named alternative costs little to keep.
- `naming-lowercase-for-values` and `naming-uppercase-for-types` cover casing conventions for named identifiers; this rule is specifically about *whether* a name should be a symbol at all.
- Overriding a symbolic operator from a well-known typeclass (`Semigroup`'s `combine`, exposed as `|+|` via cats' syntax) inherits that operator's established meaning — that is a use of convention, not an invention of a new one.

## References
- [Scala Style Guide — Method Names](https://docs.scala-lang.org/style/method-invocation.html)
