---
title: Fail the Build on Unused Imports with -Wunused, Not the Retired 2.13 Flag Name
impact: MEDIUM
impactDescription: dead imports fail compilation immediately on Scala 3, instead of a stale -Ywarn-unused flag silently doing nothing
tags: [compiler-flags, wunused, scala3, unused-imports]
---

# Fail the Build on Unused Imports with -Wunused, Not the Retired 2.13 Flag Name [MEDIUM]

## Description
Scala 3 renamed the unused-code warning family from Scala 2.13's `-Ywarn-unused`/`-Ywarn-unused:imports,privates,locals,params` to `-Wunused`, with the same per-category granularity: `-Wunused:imports`, `-Wunused:privates`, `-Wunused:locals`, `-Wunused:params`, `-Wunused:implicits`, and the shorthand `-Wunused:all` enabling every category at once. A project migrated from Scala 2.13 that kept the old `-Ywarn-unused` flag name in `scalacOptions` doesn't get an error calling out the stale flag on every Scala 3 toolchain — the practical effect is that unused-import detection quietly stops happening, and nothing in the build output calls attention to why.

`-Wunused:imports` alone only warns; combine it with `-Xfatal-warnings` (see [`compiler-flags-xfatal-warnings`](compiler-flags-xfatal-warnings.md)) — or the narrower `-Wconf:cat=unused:e` if other warning categories should stay non-fatal — to actually fail the build on a dead import rather than merely reporting it. `RemoveUnused` in Scalafix reads these same diagnostics to decide what it's allowed to delete (see [`scalafix-removeunused-rule`](scalafix-removeunused-rule.md)), so this flag isn't only a warning source — it's also a prerequisite for that rule's automated pruning.

## Bad Example
```scala
// build.sbt — stale Scala 2.13 flag name on a Scala 3 project; does nothing
scalacOptions += "-Ywarn-unused:imports"
```

## Good Example
```scala
// build.sbt
scalacOptions ++= Seq("-Wunused:imports", "-Xfatal-warnings")
```

## Notes
- `-Wunused:all` (available since Scala 3.3.0) enables every unused-code category in one flag, equivalent to listing `imports,privates,locals,params,implicits` explicitly.
- Verify any flag copied from a Scala 2.13 build against the current Scala 3 options reference before assuming it still applies — several `-Y`-prefixed 2.13 flags were renamed to `-W`-prefixed ones, not merely carried over.
- A build with mixed 2.13/3.x cross-compilation (see [`build-cross-scala-version-support`](build-cross-scala-version-support.md)) needs both flag spellings, scoped per Scala version via `scalacOptions := (if (scalaVersion.value.startsWith("3")) ... else ...)`.

## References
- [Scala 3 — Compiler Options: Scala 2 vs Scala 3](https://docs.scala-lang.org/scala3/guides/migration/options-lookup.html)
