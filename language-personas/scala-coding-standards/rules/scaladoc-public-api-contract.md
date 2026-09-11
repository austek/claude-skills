---
title: Document the Contract, Not a Restatement of the Implementation
impact: MEDIUM
impactDescription: lets a caller use a public method correctly from its doc comment alone, with no need to read the body
tags: [scaladoc, documentation, api-design]
---

# Document the Contract, Not a Restatement of the Implementation [MEDIUM]

## Description
A doc comment on a public method exists to answer the questions its signature can't: what does this method promise to a caller who never reads its body? A one-line paraphrase of the method name — `/** Charges the given amount. */` on `def charge(amount: BigDecimal): Either[PaymentError, Receipt]` — answers nothing the signature didn't already say, and leaves out everything the signature *can't* express: which conditions produce which `Left` case, whether the operation is safe to retry, whether it has side effects beyond its return value. A caller reading only the doc comment (which is the point of having one — nobody should have to read a method's implementation to use it correctly) gets no more information from that comment than from the name alone.

Write the doc comment around the contract instead: preconditions the caller must satisfy, the meaning of each distinct failure case in the return type, and any side effect or ordering requirement not visible in the signature. For a method returning `Either[E, A]` or throwing under specific conditions, enumerate what each outcome means — this is exactly the information `error-handling-typed-errors-over-strings`'s typed error cases carry in the type, but the type alone doesn't say *when* each case occurs, which is what the doc comment is for. A private, internal-only method rarely needs this treatment — its "callers" are the few lines around it, already visible in the same file — the discipline matters most at a public boundary where the implementation is not expected to be read.

## Bad Example
```scala
/** Charges the given amount. */
def charge(amount: BigDecimal): Either[PaymentError, Receipt] = ???
```

## Good Example
```scala
/** Charges `amount` against the configured payment gateway.
  *
  * @return `Left(PaymentError.InsufficientFunds)` when the gateway declines the charge,
  *         `Left(PaymentError.GatewayUnavailable)` when the gateway cannot be reached,
  *         `Right(receipt)` otherwise.
  */
def charge(amount: BigDecimal): Either[PaymentError, Receipt] = ???
```

## Notes
- `scaladoc-no-signature-narration` covers the companion mistake — restating parameter types and names the signature already shows, rather than restating the method name; both are failures to add information beyond what's already visible.
- A method whose full contract is genuinely captured by its name and type signature (`def isEmpty: Boolean`) doesn't need a doc comment at all — the contract discipline is about earning the comment's presence, not mandating one on every public member regardless of whether it says anything new.
- `@throws` documents an exception a caller must be prepared to catch, the same way the `@return` cases above document each `Left` a caller must be prepared to branch on.
- This is the same "comment the consequence, not the visible args" discipline applied to doc comments specifically, where the audience is every future caller rather than the next line of code.

## References
- [Scaladoc for Library Authors](https://docs.scala-lang.org/style/scaladoc.html)
