---
title: Always Enable -deprecation and -feature, Not Just the Warning Count
impact: MEDIUM
impactDescription: every deprecated call and feature-gated construct gets a file:line, instead of an opaque "N warnings, re-run for details" summary
tags: [compiler-flags, deprecation, feature, scalac]
---

# Always Enable -deprecation and -feature, Not Just the Warning Count [MEDIUM]

## Description
Without `-deprecation`, the compiler still notices deprecated API usage, but only reports a terse end-of-run summary — "N deprecations; re-run with -deprecation for details" — with no file or line attached to any individual warning, which makes the information practically useless for actually finding and fixing the calls. `-deprecation` turns each one into a normal warning pointing at its exact source location and includes the deprecation message the API author wrote (often naming the replacement to migrate to).

`-feature` does the identical thing for a different category: uses of a "feature-gated" language construct — implicit conversions, postfix operator notation, existential types — that technically compile but are considered advanced enough to warrant an explicit `import scala.language.X` acknowledging the choice. Left off, these also collapse into a bare count with no locations. Both flags are pure information-density improvements over what the compiler already computes — they don't change which programs compile, only how much detail accompanies a warning that already exists — so there's no adoption cost to enabling them immediately, unlike `-Xfatal-warnings` which can break an existing build.

## Bad Example
```scala
// build.sbt — deprecation/feature diagnostics collapse into an opaque count
scalacOptions := Seq("-Xfatal-warnings")
// "there were 3 deprecation warnings; re-run with -deprecation for details"
```

## Good Example
```scala
// build.sbt
scalacOptions ++= Seq("-deprecation", "-feature", "-Xfatal-warnings")
```

## Notes
- Enable both regardless of whether `-Xfatal-warnings` is also on — they're worth having even in warning-only mode, purely for the added detail.
- Combine with [`compiler-flags-xfatal-warnings`](compiler-flags-xfatal-warnings.md)'s `-Wconf` carve-outs to silence a specific noisy deprecation (a dependency's API) while still seeing the rest with full detail.
- A missing `-deprecation` is easy to overlook precisely because the build still "works" — the summary line appears in build output but is routinely ignored since it names no actionable location.

## References
- [Scala 3 — Compiler Options Reference](https://docs.scala-lang.org/scala3/guides/migration/options-new.html)
