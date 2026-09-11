---
title: Centralize Dependency Versions in project/Dependencies.scala
impact: HIGH
impactDescription: one file changes a shared library's version everywhere instead of editing every subproject's libraryDependencies by hand
tags: [sbt, dependency-management, build-configuration]
---

# Centralize Dependency Versions in project/Dependencies.scala [HIGH]

## Description
Files placed directly under `project/` are part of sbt's meta-build: sbt compiles them and their top-level `object`s become visible, unqualified, inside `build.sbt` itself — no explicit import mechanism needed beyond `import Dependencies._` at the top of `build.sbt`. Putting every library coordinate and version in one `project/Dependencies.scala` object means each subproject's `libraryDependencies` line references a named constant (`catsCore`) instead of repeating a raw `"org.typelevel" %% "cats-core" % "2.10.0"` string, so bumping a version that several modules share is a one-line edit in one file instead of a grep-and-replace across every `build.sbt` block.

This also closes a drift bug that plain hardcoded strings invite: two subprojects independently pinning the same library at slightly different versions (one updated during a refactor, the other forgotten) resolve to two different runtime behaviors of "the same" dependency, and nothing about the build surfaces that they've diverged until something breaks. A single `val` shared across every reference makes divergence structurally impossible — there's only one version to read.

## Bad Example
```scala
// core/build.sbt (inline in each subproject)
libraryDependencies += "org.typelevel" %% "cats-effect" % "3.5.4"

// api/build.sbt
libraryDependencies += "org.typelevel" %% "cats-effect" % "3.5.3" // drifted, nobody noticed
```

## Good Example
```scala
// project/Dependencies.scala
import sbt._

object Dependencies {
  val catsEffectVersion = "3.5.4"
  val catsEffect: ModuleID = "org.typelevel" %% "cats-effect" % catsEffectVersion
}
```
```scala
// build.sbt
import Dependencies._

lazy val core = (project in file("core")).settings(libraryDependencies += catsEffect)
lazy val api  = (project in file("api")).settings(libraryDependencies += catsEffect)
```

## Notes
- Group related coordinates that always travel together into a `Seq[ModuleID]` (a "bundle") so adding or removing one updates every module that references the bundle.
- Works together with [`build-cross-scala-version-support`](build-cross-scala-version-support.md): the `%%` operator in these constants still resolves the correct binary-version suffix per cross-build.
- For enforcing a single resolved version across transitive dependencies too (not just direct ones), pair this with `dependencyOverrides` or an sbt-managed BOM-style enforcement.

## References
- [sbt Reference — Organizing the build](https://www.scala-sbt.org/1.x/docs/Organizing-Build.html)
