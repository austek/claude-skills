---
title: Pair -Xfatal-warnings with -Wconf Instead of an All-or-Nothing Switch
impact: HIGH
impactDescription: warnings fail the build category by category, so one unrelated deprecation doesn't block every PR while the team catches up
tags: [compiler-flags, xfatal-warnings, wconf, scala3]
---

# Pair -Xfatal-warnings with -Wconf Instead of an All-or-Nothing Switch [HIGH]

## Description
`-Xfatal-warnings` turns every compiler warning into a hard error — a clean way to stop new warnings from accumulating, but blunt on its own: the first warning from any category (a deprecation in a dependency's API, an unchecked pattern match) fails the whole compile, with no way to let that one category through while still failing on everything else. `-Wconf` (Scala 2.13.2+ and Scala 3) filters warnings by category, source location, or message pattern and assigns each match an action — `error`, `warning`, or `silent` — so `-Xfatal-warnings` can be scoped down to "fatal except for the categories I've explicitly chosen to downgrade" instead of "fatal, full stop."

`-Wconf:cat=deprecation:s,any:e` reads as two ordered filters: deprecation warnings are silenced (`s`), and everything else (`any`) is an error (`e`) — filters are evaluated in order, so a specific carve-out has to come before the catch-all `any` rule it's meant to override. This is what makes adopting `-Xfatal-warnings` on an existing codebase practical: enable it globally, then add one `-Wconf` filter per category that still has legitimate pre-existing warnings, shrinking that filter list as the code gets cleaned up.

## Bad Example
```scala
// build.sbt — one warning anywhere, in any category, blocks every build
scalacOptions += "-Xfatal-warnings"
```

## Good Example
```scala
// build.sbt — fatal by default, with a deliberate, visible carve-out
scalacOptions ++= Seq(
  "-Xfatal-warnings",
  "-Wconf:cat=deprecation:s"
)
```

## Notes
- Filter keys include `cat` (category), `site` (a regex over the enclosing symbol), `origin` (a regex over the source of the warning), and `msg` (a regex over the warning text) — combine several to scope a carve-out precisely.
- This same selective-fatality pattern is the compiler-flag equivalent of gating wart/style-check adoption incrementally — see [`wartremover-scalastyle-baseline-for-legacy-code`](wartremover-scalastyle-baseline-for-legacy-code.md).
- Treat `-Wconf` carve-outs as temporary and tracked, not permanent configuration — a shrinking exemption list is the signal that adoption is progressing.

## References
- [Scala 3 — Compiler Warnings and -Wconf](https://docs.scala-lang.org/scala3/guides/migration/options-lookup.html)
