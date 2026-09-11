---
title: Structure a Multi-Module sbt Build with aggregate and dependsOn
impact: HIGH
impactDescription: one `sbt compile`/`sbt test` at the root builds and tests every module in the right order, instead of manually visiting each subproject
tags: [sbt, build, multi-module, aggregate]
---

# Structure a Multi-Module sbt Build with aggregate and dependsOn [HIGH]

## Description
An sbt build splits into subprojects along two independent axes: `dependsOn` puts one project's classes on another's compile/runtime classpath (a real code dependency — `api` can't compile without `core`), while `aggregate` only fans a root-level command (`compile`, `test`, `publish`) out to the listed projects so a single invocation at the root touches all of them. A project usually needs both: `dependsOn` for the classpath, `aggregate` so `sbt compile` from the repo root doesn't silently build only the root project while every subproject sits untouched.

Pull shared settings (`scalaVersion`, `organization`, common compiler flags) into one `Seq[Def.Setting[_]]` and apply it to every subproject with `.settings(commonSettings)`. Without it, each subproject's settings block repeats the same lines, and a Scala version bump means finding and editing every one of them rather than one shared value. Give the root project `publish / skip := true` — it exists to aggregate, not to ship an artifact of its own.

## Bad Example
```scala
// build.sbt — no aggregate, no shared settings
lazy val core = (project in file("core"))
  .settings(scalaVersion := "3.3.3", organization := "com.example")

lazy val api = (project in file("api"))
  .dependsOn(core)
  .settings(scalaVersion := "3.3.4", organization := "com.example") // drifted, unnoticed
```

## Good Example
```scala
// build.sbt
lazy val commonSettings = Seq(
  scalaVersion := "3.3.3",
  organization := "com.example"
)

lazy val core = (project in file("core"))
  .settings(commonSettings)
  .settings(name := "myapp-core")

lazy val api = (project in file("api"))
  .dependsOn(core)
  .settings(commonSettings)
  .settings(name := "myapp-api")

lazy val root = (project in file("."))
  .aggregate(core, api)
  .settings(publish / skip := true)
```

## Notes
- `dependsOn(core % "test->test")` also shares `core`'s test sources (e.g. shared fixtures) with a dependent module's tests.
- Centralize dependency coordinates too, not just `scalaVersion` — see [`build-dependency-version-management`](build-dependency-version-management.md).
- A module needed only for its test scope (`dependsOn(core % Test)`) keeps it off the dependent module's compile classpath.

## References
- [sbt Reference — Multi-project builds](https://www.scala-sbt.org/1.x/docs/Multi-Project.html)
