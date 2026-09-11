---
title: Cross-Build with crossScalaVersions and the + Command, Not Duplicated Modules
impact: MEDIUM
impactDescription: `sbt +test` runs the whole suite against every supported Scala version in one command instead of a hand-maintained matrix
tags: [sbt, cross-build, crossScalaVersions, scala-version]
---

# Cross-Build with crossScalaVersions and the + Command, Not Duplicated Modules [MEDIUM]

## Description
`crossScalaVersions := Seq("2.13.14", "3.3.3")` declares which Scala versions a module publishes for; `scalaVersion` still picks the version used for an ordinary (non-cross) command, conventionally set to `crossScalaVersions.value.head`. Prefixing any task with `+` — `sbt +compile`, `sbt +test`, `sbt +publish` — reruns it once per version in `crossScalaVersions`, so the whole matrix is exercised from one invocation instead of manually switching versions and re-running by hand.

The `%%` operator (`"org.typelevel" %% "cats-core" % "2.10.0"`) is what makes this safe: it appends the *binary* Scala version of the current cross-build run to the artifact name at resolution time, so the same dependency declaration resolves `cats-core_2.13` under a 2.13 run and `cats-core_3` under a 3.x run. Writing the suffix by hand with plain `%` (`"cats-core_2.13"`) hardcodes one version and silently breaks the moment `+compile` runs against the other one — the artifact doesn't exist under that name for that Scala version, and resolution fails with a missing-dependency error rather than a clear "wrong version" message.

## Bad Example
```scala
// build.sbt — no crossScalaVersions; two near-duplicate projects instead
lazy val core213 = (project in file("core")).settings(scalaVersion := "2.13.14")
lazy val core3   = (project in file("core")).settings(scalaVersion := "3.3.3") // same sources, redeclared
```

## Good Example
```scala
// build.sbt
lazy val core = (project in file("core"))
  .settings(
    crossScalaVersions := Seq("2.13.14", "3.3.3"),
    scalaVersion := crossScalaVersions.value.head,
    libraryDependencies += "org.typelevel" %% "cats-core" % "2.10.0"
  )
```
```bash
sbt +test      # runs the suite under 2.13.14, then under 3.3.3
```

## Notes
- `crossScalaVersions` must be set on every module that's part of the cross-build; a module missing it just runs once, under `scalaVersion`, on every `+` invocation.
- Mill's cross-build equivalent uses `Cross[M]("2.13.14", "3.3.3")` — see [`build-mill-build-definition`](build-mill-build-definition.md).
- CI matrices that shell out per-version instead of using `+` duplicate this logic outside the build tool and drift from `crossScalaVersions` over time.

## References
- [sbt Reference — Cross-building](https://www.scala-sbt.org/1.x/docs/Cross-Build.html)
