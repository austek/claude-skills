---
title: Scope Wartremover to Compile, Not Test, and Let It Fail via Errors Not Warnings
impact: MEDIUM
impactDescription: violations fail the ordinary compile step with no extra CI task to configure, and test code keeps the mutability it legitimately needs
tags: [wartremover, ci, compiler-plugin, sbt]
---

# Scope Wartremover to Compile, Not Test, and Let It Fail via Errors Not Warnings [MEDIUM]

## Description
Wartremover being a compiler plugin, rather than a standalone linter task, changes how it fits into CI compared to Scalafmt or Scalafix: there's no separate `wartremoverCheck` invocation to add to a pipeline, because a wart configured under `wartremoverErrors` already fails `sbt compile` itself — any CI job that already runs `compile` or `test` gets the enforcement for free. The only lever CI-side is which setting scopes it: `wartremoverErrors` fails the build on a match, `wartremoverWarnings` only prints. Using `Warnings` in a CI job defeats the purpose the same way running `scalafmtAll` instead of `scalafmtCheckAll` does — the message appears in build output, but nothing stops the merge.

Test code is a legitimate exception to some warts: mutable `var`s in test fixtures for accumulating call counts, or a test-only `null` to exercise a null-handling code path, are reasonable and shouldn't fail the build. Scoping `wartremoverErrors` to `Compile / compile` only, and leaving `Test / compile` unrestricted (or restricted to a smaller wart set), keeps enforcement on production code without fighting legitimate test patterns.

## Bad Example
```scala
// build.sbt — Warnings never fails the build; nothing blocks a merge
Compile / compile / wartremoverWarnings ++= Seq(Wart.Null, Wart.Var, Wart.Any)
```

## Good Example
```scala
// build.sbt
Compile / compile / wartremoverErrors ++= Seq(Wart.Null, Wart.Var, Wart.Any)
Test / compile / wartremoverErrors := Seq.empty
```

## Notes
- No `--check` flag or dedicated CI step is needed beyond whatever already invokes `compile`/`test` — this is the main practical difference from [`scalafix-ci-check-mode`](scalafix-ci-check-mode.md) and [`scalafmt-ci-check-mode`](scalafmt-ci-check-mode.md).
- `wartremoverExcluded` scopes exclusion by file path instead of by configuration, useful for a generated-sources directory that can't satisfy the warts.
- Pin the plugin version the same way [`scalafmt-config-committed`](scalafmt-config-committed.md) pins scalafmt's — an unpinned wartremover version can add new warts to an existing `Wart.all`-style list without warning.

## References
- [Wartremover — sbt Plugin Usage](https://www.wartremover.org/doc/install-setup.html)
