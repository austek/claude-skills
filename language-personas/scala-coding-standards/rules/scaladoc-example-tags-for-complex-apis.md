---
title: Show a Complex Generic API's Usage With @example, Not Prose Alone
impact: LOW
impactDescription: lets a caller copy a working call shape instead of reverse-engineering one from a type signature and a paragraph
tags: [scaladoc, documentation, examples]
---

# Show a Complex Generic API's Usage With @example, Not Prose Alone [LOW]

## Description
A method with several type parameters, curried argument lists, or context-dependent (`using`) parameters can have a doc comment that correctly and completely describes its contract in prose and still leave a reader unsure what an actual call looks like — `def combine[A](a: Option[A], b: Option[A])(f: (A, A) => A): Option[A]`'s behavior ("combines two optional values with the given function, propagating absence") is accurate, but a reader still has to mentally assemble the call syntax, the curried parameter grouping, and what "propagating absence" produces for a concrete pair of inputs. This gap grows with genuinely generic or type-class-parameterized APIs, where the type signature alone (`using Encoder[T]`, several type parameters interacting) can be harder to parse than the one line of code that would show a working example.

Scaladoc's `@example` tag exists for exactly this: a fenced `{{{ ... }}}` block showing a real call, ideally with the result alongside it as a comment, right below the prose description. This isn't a replacement for the contract description covered by `scaladoc-public-api-contract` — it's an addition for the specific case where prose and a type signature together still leave the call shape non-obvious. Reach for it on generic, type-class-heavy, or multi-parameter-list APIs; a simple method with an obvious call shape (`def isEmpty: Boolean`) gains nothing from an example and doesn't need one.

## Bad Example
```scala
/** Combines two optional values using the given binary function, propagating `None` if either is absent. */
def combine[A](a: Option[A], b: Option[A])(f: (A, A) => A): Option[A] =
  for x <- a; y <- b yield f(x, y)
```

## Good Example
```scala
/** Combines two optional values using the given binary function, propagating `None` if either is absent.
  *
  * @example {{{
  *   combine(Some(2), Some(3))(_ + _) // Some(5)
  *   combine(None, Some(3))(_ + _)    // None
  * }}}
  */
def combine[A](a: Option[A], b: Option[A])(f: (A, A) => A): Option[A] =
  for x <- a; y <- b yield f(x, y)
```

## Notes
- `{{{ ... }}}` is Scaladoc's own fenced-code-block syntax, distinct from Markdown's triple backtick — it's what the generated HTML documentation renders as a formatted code sample.
- Keep the example minimal and runnable-looking — one or two calls with their result shown as a trailing comment communicates more, faster, than a longer scenario.
- This tag earns its place on genuinely non-obvious call shapes; adding `@example` to every method regardless of complexity just adds noise the reader has to skip past to find the ones that matter.
- `scaladoc-no-signature-narration` still applies inside the surrounding prose — the `@example` block carries the call shape; the prose above it should still describe the contract, not restate the signature.

## References
- [Scaladoc for Library Authors — Comment Tags](https://docs.scala-lang.org/style/scaladoc.html)
