---
title: Prune Unused Imports and Locals with the RemoveUnused Rule
impact: MEDIUM
impactDescription: unused imports and private members get deleted automatically on every scalafix run instead of accumulating until someone does a manual sweep
tags: [scalafix, removeunused, unused-imports]
---

# Prune Unused Imports and Locals with the RemoveUnused Rule [MEDIUM]

## Description
`RemoveUnused` is a semantic scalafix rule that deletes code the compiler has already flagged as unused — unused imports, private members, local `val`s, and (opt-in) unused parameters — rather than re-implementing that detection itself. It acts on the compiler's own `-Wunused` (Scala 3) / `-Ywarn-unused` (Scala 2.13) diagnostics, so those warnings must be enabled in `scalacOptions` for a given category before `RemoveUnused` can remove anything in that category; without `-Wunused:imports`, for instance, the rule has no unused-import warnings to act on and leaves imports untouched.

Each category is toggled independently in `.scalafix.conf`, which matters because removing unused parameters is riskier than removing unused imports — a parameter can be unused inside a method while still being part of a public API contract that removing it would break — so it's common to enable `imports`, `privates`, and `locals` but leave `params` off.

## Bad Example
```scala
// no automated pruning — dead imports accumulate silently after refactors
import scala.concurrent.Future
import scala.util.Try // no longer used since the last refactor

def loadUser(id: String): Future[User] = repo.find(id)
```

## Good Example
```
# .scalafix.conf
rules = [RemoveUnused]
RemoveUnused.imports = true
RemoveUnused.privates = true
RemoveUnused.locals = true
RemoveUnused.params = false
```
```scala
// build.sbt
scalacOptions ++= Seq("-Wunused:imports,privates,locals")
```

## Notes
- Requires SemanticDB — see [`scalafix-semantic-rules-require-semanticdb`](scalafix-semantic-rules-require-semanticdb.md).
- Run via `sbt scalafixAll` locally to apply fixes, and `sbt "scalafixAll --check"` on CI to fail the build if anything would be removed — see [`scalafix-ci-check-mode`](scalafix-ci-check-mode.md).
- `RemoveUnused.params` is worth enabling only inside `private`/`local` methods, since it's semantic-rule scope, not visibility-aware by itself.

## References
- [Scalafix — RemoveUnused Rule](https://scalacenter.github.io/scalafix/docs/rules/RemoveUnused.html)
