---
title: Add Explicit Result Types with the ExplicitResultTypes Rule
impact: MEDIUM
impactDescription: public API signatures document their own return type instead of requiring a reader (or a downstream compile) to re-derive it via inference
tags: [scalafix, explicitresulttypes, api-design, type-inference]
---

# Add Explicit Result Types with the ExplicitResultTypes Rule [MEDIUM]

## Description
`ExplicitResultTypes` is a semantic scalafix rule that inserts the compiler's inferred return type as an explicit annotation on public, non-local members that lack one, then leaves it there as ordinary source code. Two problems come from leaving return types inferred on a public API: a reader has to mentally re-run type inference to know what a member returns, and — more concretely — a change to that member's implementation can silently widen or change its inferred type, which downstream callers (and binary-compatibility tooling like MiMa) only discover once something fails to compile against the new signature instead of catching it as a deliberate, reviewable diff on the member itself.

Explicit types on public members also make separate compilation cheaper: a caller in another module needs only the signature to type-check a call site, whereas an inferred type forces the compiler to have already fully elaborated the defining module before it can determine what's on the other side of the call.

## Bad Example
```scala
object PriceCalculator {
  def discount(price: BigDecimal) = price * BigDecimal("0.9")
  def defaults = Map("retries" -> 3, "timeoutMs" -> 5000)
}
```

## Good Example
```scala
object PriceCalculator {
  def discount(price: BigDecimal): BigDecimal = price * BigDecimal("0.9")
  def defaults: Map[String, Int] = Map("retries" -> 3, "timeoutMs" -> 5000)
}
```

## Notes
- Requires SemanticDB — see [`scalafix-semantic-rules-require-semanticdb`](scalafix-semantic-rules-require-semanticdb.md).
- Scoped to public members by default; `ExplicitResultTypes.memberVisibility = [Public, Protected]` widens it to protected members too.
- Run once against an existing codebase as a one-time bulk fix (`sbt scalafixAll`), then rely on [`scalafix-ci-check-mode`](scalafix-ci-check-mode.md) to keep new public members annotated going forward.

## References
- [Scalafix — ExplicitResultTypes Rule](https://scalacenter.github.io/scalafix/docs/rules/ExplicitResultTypes.html)
