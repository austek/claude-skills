---
title: Replace Nested flatMap Chains With a For-Comprehension
impact: MEDIUM
impactDescription: removes one indent level per step and stops separating a binding from its use
tags: [for-comprehension, readability, flatmap]
---

# Replace Nested flatMap Chains With a For-Comprehension [MEDIUM]

## Description
Composing three or more dependent monadic steps by hand — `.flatMap(a => ...flatMap(b => ...map(c => ...)))` — pushes each subsequent step one indent level to the right, a shape often called the "pyramid of doom." Every nested lambda also separates a bound name from the line that uses it: `a` is bound at the outermost call but only read several closures deeper, so the reader has to hold open brackets in their head to see which variable belongs to which scope. None of this changes what the code does — a `for`-comprehension desugars to exactly the same sequence of `flatMap`/`map` calls (`for-comprehensions-monadic-composition`) — so switching to it is a pure readability win with no runtime cost.

This is not a blanket objection to `flatMap`: a single `flatMap` composed with a single `map`, or a `flatMap` used as one link in a larger `.filter`/`.foreach` pipeline, is often clearer written directly. The failure mode this rule targets is specifically the accumulation of several *dependent* steps — each needing the previous step's result — expressed as nested calls instead of the sequential syntax built for exactly that shape. As a rule of thumb, once a third dependent step gets added to a hand-written chain, that is the point to flatten it into a `for`-comprehension rather than nesting further.

## Bad Example
```scala
def checkout(cartId: String): Either[CheckoutError, Receipt] =
  findCart(cartId).flatMap { cart =>
    reserveInventory(cart).flatMap { reservation =>
      chargePayment(cart, reservation).map { charge =>
        Receipt(cart, reservation, charge)
      }
    }
  }
```

## Good Example
```scala
def checkout(cartId: String): Either[CheckoutError, Receipt] =
  for
    cart        <- findCart(cartId)
    reservation <- reserveInventory(cart)
    charge      <- chargePayment(cart, reservation)
  yield Receipt(cart, reservation, charge)
```

## Notes
- Both forms compile to the identical `flatMap`/`flatMap`/`map` call sequence; the rewrite is for the reader, not the runtime.
- One `flatMap` feeding one `map` rarely benefits from a `for`-comprehension wrapper — see `for-comprehensions-readability-over-cleverness` for when the sugar adds ceremony instead of removing it.
- The same nesting problem shows up identically for `Future`, `Try`, and effect types like `IO`; the fix is the same regardless of which type is being composed.
- Renaming each bound value to something more specific than the generic `a`/`b`/`c` seen in hand-nested code is a side benefit — a `for`-comprehension's flat layout leaves room to name each step's result for what it actually is (`cart`, `reservation`, `charge`).

## References
- [Scala 3 Book — For Expressions](https://docs.scala-lang.org/scala3/book/control-structures.html#for-expressions)
