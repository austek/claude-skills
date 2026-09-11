---
title: Define Mill Modules with ScalaModule and moduleDeps
impact: MEDIUM
impactDescription: a build file that reads as plain Scala instead of sbt's separate settings-key DSL, with faster incremental builds via Mill's fine-grained target caching
tags: [mill, build, scalamodule, moduleDeps]
---

# Define Mill Modules with ScalaModule and moduleDeps [MEDIUM]

## Description
Mill's build file (`build.mill` in current releases, `build.sc` in older ones) is plain Scala: a module is an `object` extending `ScalaModule`, and its build parameters — `scalaVersion`, `ivyDeps` — are ordinary overridable `def`s rather than sbt's `key := value` settings-key mechanism. `ivyDeps` takes an `Agg` of `ivy"group::artifact:version"` interpolated strings, using `::` for a Scala-suffixed artifact the same way sbt's `%%` appends the binary Scala version.

Inter-module dependencies are declared with `def moduleDeps = Seq(core)`, Mill's equivalent of sbt's `dependsOn` — it puts `core`'s compiled output on `api`'s compile classpath and, because Mill resolves per-target dependency graphs directly rather than through separate aggregate/dependsOn axes, a single `mill __.compile` (or `mill api.compile`, which pulls in `core` automatically) covers the whole dependency chain with no separate aggregation step to remember.

Each `def` is a cached target: Mill only reruns the targets whose inputs actually changed, computed from the module graph itself rather than a file-timestamp heuristic layered on top.

## Bad Example
```scala
// build.mill — duplicated scalaVersion, no moduleDeps: api can't see core's classes
object core extends ScalaModule {
  def scalaVersion = "3.3.3"
}

object api extends ScalaModule {
  def scalaVersion = "3.3.4" // drifted from core
  def ivyDeps = Agg(ivy"com.example::myapp-core:1.0.0") // published dep instead of a module link
}
```

## Good Example
```scala
// build.mill
package build
import mill._, scalalib._

trait Common extends ScalaModule {
  def scalaVersion = "3.3.3"
}

object core extends Common {
  def ivyDeps = Agg(ivy"org.typelevel::cats-core:2.10.0")
}

object api extends Common {
  def moduleDeps = Seq(core)
}
```

## Notes
- A shared `trait Common extends ScalaModule` plays the same role as sbt's `commonSettings` — override once, mix into every module.
- Mill's cross-version builds use `Cross[M]("2.13.14", "3.3.3")` with `trait M extends Cross.Module[String] with ScalaModule { def scalaVersion = crossValue }`; see [`build-cross-scala-version-support`](build-cross-scala-version-support.md) for the sbt equivalent.
- `def ivyDeps` returning a fixed `Agg` is the common case; it can also be a `T[Agg[Dep]]` task when a dependency set is computed from another target.

## References
- [Mill — Scala Module Reference](https://mill-build.org/mill/scalalib/module-config.html)
