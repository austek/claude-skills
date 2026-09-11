# Section Definitions

## Build (build)
**Impact:** HIGH

Configure sbt and Mill builds for correct module layout, cross-building, and centralized dependency versions.

**Rules:**
- `build-cross-scala-version-support` - Cross-build with crossScalaVersions and the + command, not duplicated modules
- `build-dependency-version-management` - Centralize dependency versions in project/Dependencies.scala
- `build-mill-build-definition` - Define Mill modules with ScalaModule and moduleDeps
- `build-sbt-multi-module-layout` - Structure a multi-module sbt build with aggregate and dependsOn

## Compiler Flags (compiler-flags)
**Impact:** HIGH

Turn on the Scala compiler's warning and strictness flags deliberately, and gate CI on them without an all-or-nothing switch.

**Rules:**
- `compiler-flags-deprecation-and-feature-warnings` - Always enable -deprecation and -feature, not just the warning count
- `compiler-flags-scala3-strict-mode` - Assemble a strict Scala 3 flag profile beyond the defaults
- `compiler-flags-unused-imports-error` - Fail the build on unused imports with -Wunused, not the retired 2.13 flag name
- `compiler-flags-xfatal-warnings` - Pair -Xfatal-warnings with -Wconf instead of an all-or-nothing switch

## Scalafix (scalafix)
**Impact:** HIGH

Run Scalafix's syntactic and semantic rules in CI check mode to keep the codebase free of unused code and stale patterns.

**Rules:**
- `scalafix-ci-check-mode` - Compile first, then run scalafixAll --check on CI
- `scalafix-explicitresulttypes-rule` - Add explicit result types with the ExplicitResultTypes rule
- `scalafix-removeunused-rule` - Prune unused imports and locals with the RemoveUnused rule
- `scalafix-semantic-rules-require-semanticdb` - Enable SemanticDB before running Scalafix's semantic rules

## Scalafmt (scalafmt)
**Impact:** HIGH

Commit a pinned Scalafmt config and enforce it in CI check mode, never write mode.

**Rules:**
- `scalafmt-ci-check-mode` - Run Scalafmt in check mode on CI, never in write mode
- `scalafmt-config-committed` - Commit .scalafmt.conf with a pinned version key
- `scalafmt-import-sorting` - Sort and group imports with the Imports rewrite rule
- `scalafmt-max-column-consistency` - Pick one maxColumn value and keep the IDE reading the same config

## Wartremover (wartremover)
**Impact:** HIGH

Ban unsafe patterns at compile time with Wartremover, scoped and baselined so it doesn't block legacy code.

**Rules:**
- `wartremover-ban-null-var-any` - Ban Wart.Null, Wart.Var, and Wart.Any at compile time
- `wartremover-ci-integration` - Scope Wartremover to compile, not test, and let it fail via errors not warnings
- `wartremover-scalastyle-baseline-for-legacy-code` - Baseline Wartremover and Scalastyle instead of enforcing everywhere on day one

## Scalastyle (scalastyle)
**Impact:** LOW

Gate line length and naming style with a committed scalastyle-config.xml.

**Rules:**
- `scalastyle-line-length-and-style-checks` - Gate line length and naming style with scalastyle-config.xml
