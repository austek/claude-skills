---
title: Enable SemanticDB Before Running Scalafix's Semantic Rules
impact: HIGH
impactDescription: semantic rules (RemoveUnused, ExplicitResultTypes) fail outright without this, instead of quietly doing nothing
tags: [scalafix, semanticdb, sbt-scalafix, configuration]
---

# Enable SemanticDB Before Running Scalafix's Semantic Rules [HIGH]

## Description
Scalafix rules split into two kinds: syntactic rules (`DisableSyntax`, `NoValInForComprehension`) work directly off the parse tree and need nothing extra, while semantic rules (`RemoveUnused`, `ExplicitResultTypes`, `OrganizeImports`) need type and symbol information the parser alone doesn't have — which method a call resolves to, whether an import is actually used, what a value's inferred type is. That information comes from SemanticDB, metadata the compiler emits during a real compilation and writes under `target/scala-<version>/meta` (or `META-INF/semanticdb`) as `.semanticdb` files, one per source file.

The sbt-scalafix plugin turns this on with `ThisBuild / semanticdbEnabled := true` and `ThisBuild / semanticdbVersion := scalafixSemanticdb.revision`, which pins the SemanticDB compiler plugin to the exact version scalafix expects. Because the data comes from a compile, a semantic rule needs `sbt compile` to have run first with these settings in place — running `scalafix` against stale or absent SemanticDB output fails with a "SemanticDB not found" error rather than silently skipping the rule.

## Bad Example
```scala
// build.sbt — .scalafix.conf lists RemoveUnused, but nothing emits SemanticDB
ThisBuild / scalaVersion := "3.3.3"
// sbt scalafixAll fails: "SemanticDB not found for ..."
```

## Good Example
```scala
// build.sbt
ThisBuild / scalaVersion := "3.3.3"
ThisBuild / semanticdbEnabled := true
ThisBuild / semanticdbVersion := scalafixSemanticdb.revision
```
```scala
// project/plugins.sbt
addSbtPlugin("ch.epfl.scala" % "sbt-scalafix" % "0.12.1")
```

## Notes
- On Scala 2, semantic rules additionally need `-Yrangepos`; the sbt-scalafix plugin adds it automatically once `semanticdbEnabled` is set, so it rarely needs to be written by hand.
- Run `sbt compile` before `scalafixAll` in any script or CI job — semantic rules read SemanticDB output from the most recent compile, not from source files directly.
- [`scalafix-removeunused-rule`](scalafix-removeunused-rule.md) and [`scalafix-explicitresulttypes-rule`](scalafix-explicitresulttypes-rule.md) are both semantic rules and depend on this setting.

## References
- [Scalafix — Installation](https://scalacenter.github.io/scalafix/docs/users/installation.html)
