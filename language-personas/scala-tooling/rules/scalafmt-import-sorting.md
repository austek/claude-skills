---
title: Sort and Group Imports with the Imports Rewrite Rule
impact: LOW
impactDescription: import order stops depending on the order a developer happened to type them in, removing one more class of unrelated diff noise
tags: [scalafmt, imports, rewrite-rules]
---

# Sort and Group Imports with the Imports Rewrite Rule [LOW]

## Description
Scalafmt only reorders imports when `rewrite.rules` explicitly includes `Imports` — by default it formats each import statement's internal spacing and wrapping but leaves the statements in whatever order the file already has them, so two files edited by different people drift into different, arbitrary import orders. Adding `Imports` to `rewrite.rules` turns on `rewrite.imports.*` settings: `rewrite.imports.sort` picks the ordering (`ascii`, `original` unchanged-order, or `scalastyle` — case-insensitive, matching the common Scalastyle import-order convention), and `rewrite.imports.groups` partitions imports into ordered blocks by regex, inserting a blank line between blocks.

This is one of the lower-stakes scalafmt rewrites — it changes cosmetic ordering only, never behavior — but it removes a small, constant source of unrelated diff lines: without it, a contributor who adds one import next to unrelated existing ones in a different order than a teammate would have produces a diff that's harder to scan for the actual change.

## Bad Example
```
# .scalafmt.conf — no Imports rule; order is whatever the file already has
version = "3.8.3"
runner.dialect = scala3
```
```scala
import scala.concurrent.Future
import java.time.Instant
import cats.effect.IO
import scala.util.Try
```

## Good Example
```
# .scalafmt.conf
version = "3.8.3"
runner.dialect = scala3
rewrite.rules = [Imports]
rewrite.imports.sort = scalastyle
rewrite.imports.groups = [
  ["java\\..*", "javax\\..*"],
  ["scala\\..*"],
  [".*"]
]
```
```scala
import java.time.Instant

import scala.concurrent.Future
import scala.util.Try

import cats.effect.IO
```

## Notes
- `rewrite.imports.expand = true` forces one import per line, expanding `import scala.{Left, Right}` into two statements — useful alongside grouping for consistent diffs on import changes.
- `Imports` supersedes the older `SortImports`/`AsciiSortImports`/`ExpandImportSelectors` rules, which are deprecated in current scalafmt releases.
- Combine with [`scalafmt-max-column-consistency`](scalafmt-max-column-consistency.md) — both are cosmetic rewrite settings that only pay off when applied consistently repo-wide.

## References
- [Scalafmt — Rewrite Rules](https://scalameta.org/scalafmt/docs/configuration.html#rewrite-rules)
