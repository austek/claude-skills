---
title: Don't Pile Every Dependency Into using Parameters
impact: MEDIUM
impactDescription: keeps a call site's visible argument list an honest picture of what the call actually depends on
tags: [anti-patterns, implicits, given, using]
---

# Don't Pile Every Dependency Into using Parameters [MEDIUM]

## Description
A single `using` clause requiring one well-defined type class instance (`implicits-givens-type-class-pattern`) is a precise, compiler-checked way to express "this generic function needs evidence of `T`." That precision erodes once a signature accumulates several unrelated `using` parameters — a logger, a tracer, application config, a clock, an `ExecutionContext` — each individually justified, but together turning `process(order)` at the call site into a call that silently threads five capabilities the reader can't see without opening the signature. Implicit resolution compounds the problem as more givens accumulate in scope: two `given` instances of the same (or a related) type visible at once produce an ambiguous-implicit compile error that names the conflict but not always why both instances are in scope, which can take real digging to resolve in a codebase with many small `given` definitions spread across files.

The fix is usually to stop treating "can be implicit" as the deciding question and ask instead whether the dependency is something that genuinely varies per call (a real case for a parameter, implicit or explicit) or is really a fixed bundle of collaborators a component needs for its whole lifetime — the second case is better modeled as an explicit constructor parameter or a single context object, passed once, rather than re-summoned implicitly at every method call. Reserve `using` for what it does best: type class evidence resolved structurally from the type itself, not a general-purpose way to avoid writing an explicit argument.

## Bad Example
```scala
def process(order: Order)(using logger: Logger, tracer: Tracer, config: AppConfig, clock: Clock, ec: ExecutionContext): Unit =
  logger.log(s"processing ${order.id}")
  ??? // five implicit capabilities threaded through every call site, none of them visible at process(order)
```

## Good Example
```scala
final case class OrderContext(logger: Logger, tracer: Tracer, config: AppConfig)

def process(order: Order, ctx: OrderContext): Unit =
  ctx.logger.log(s"processing ${order.id}")
```

## Notes
- `implicits-givens-type-class-pattern` covers the legitimate, narrow use this rule isn't arguing against — a single `using` clause for one type class instance, resolved structurally, stays precise no matter how many call sites use it.
- An explicit context object (`OrderContext` above) is still injectable and testable the same way implicit parameters are, just visible at the call site instead of resolved silently — nothing about explicitness here costs the flexibility implicits were reached for in the first place.
- `implicits-givens-explicit-imports-no-wildcard` and `concurrency-execution-context-explicit` both push in the same direction from different angles — narrow, visible dependency wiring over broad, ambient implicit scope.
- A genuinely ambiguous-implicit compile error is the compiler catching this anti-pattern in the act — treat it as a signal to consolidate or make a dependency explicit, not just as an obstacle to silence with an extra import.

## References
- [Scala 3 Reference — Given Instances](https://docs.scala-lang.org/scala3/reference/contextual/givens.html)
