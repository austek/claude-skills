# Scala Tooling

A comprehensive guide to Scala development tools, designed for AI agents and LLMs to configure and use modern Scala tooling effectively.

## Overview

This skill provides 20 rules across 6 categories:

| Category | Prefix | Impact | Rules |
|----------|--------|--------|-------|
| Build | `build-` | HIGH | 4 |
| Compiler Flags | `compiler-flags-` | HIGH | 4 |
| Scalafix | `scalafix-` | HIGH | 4 |
| Scalafmt | `scalafmt-` | HIGH | 4 |
| Wartremover | `wartremover-` | HIGH | 3 |
| Scalastyle | `scalastyle-` | LOW | 1 |

## Structure

```
skills/scala-tooling/
├── SKILL.md              # Skill overview with quick reference
├── metadata.json         # Metadata (version, description)
├── README.md             # This file
└── rules/
    ├── _sections.md      # Section definitions
    ├── _template.md      # Rule template
    ├── build-*.md            # Build rules
    ├── compiler-flags-*.md   # Compiler Flags rules
    ├── scalafix-*.md         # Scalafix rules
    ├── scalafmt-*.md         # Scalafmt rules
    ├── wartremover-*.md      # Wartremover rules
    └── scalastyle-*.md       # Scalastyle rules
```

## Rules

### Build (HIGH)
- `build-cross-scala-version-support` - Cross-build with crossScalaVersions and the + command, not duplicated modules
- `build-dependency-version-management` - Centralize dependency versions in project/Dependencies.scala
- `build-mill-build-definition` - Define Mill modules with ScalaModule and moduleDeps
- `build-sbt-multi-module-layout` - Structure a multi-module sbt build with aggregate and dependsOn

### Compiler Flags (HIGH)
- `compiler-flags-deprecation-and-feature-warnings` - Always enable -deprecation and -feature, not just the warning count
- `compiler-flags-scala3-strict-mode` - Assemble a strict Scala 3 flag profile beyond the defaults
- `compiler-flags-unused-imports-error` - Fail the build on unused imports with -Wunused, not the retired 2.13 flag name
- `compiler-flags-xfatal-warnings` - Pair -Xfatal-warnings with -Wconf instead of an all-or-nothing switch

### Scalafix (HIGH)
- `scalafix-ci-check-mode` - Compile first, then run scalafixAll --check on CI
- `scalafix-explicitresulttypes-rule` - Add explicit result types with the ExplicitResultTypes rule
- `scalafix-removeunused-rule` - Prune unused imports and locals with the RemoveUnused rule
- `scalafix-semantic-rules-require-semanticdb` - Enable SemanticDB before running Scalafix's semantic rules

### Scalafmt (HIGH)
- `scalafmt-ci-check-mode` - Run Scalafmt in check mode on CI, never in write mode
- `scalafmt-config-committed` - Commit .scalafmt.conf with a pinned version key
- `scalafmt-import-sorting` - Sort and group imports with the Imports rewrite rule
- `scalafmt-max-column-consistency` - Pick one maxColumn value and keep the IDE reading the same config

### Wartremover (HIGH)
- `wartremover-ban-null-var-any` - Ban Wart.Null, Wart.Var, and Wart.Any at compile time
- `wartremover-ci-integration` - Scope Wartremover to compile, not test, and let it fail via errors not warnings
- `wartremover-scalastyle-baseline-for-legacy-code` - Baseline Wartremover and Scalastyle instead of enforcing everywhere on day one

### Scalastyle (LOW)
- `scalastyle-line-length-and-style-checks` - Gate line length and naming style with scalastyle-config.xml

## Tools Covered

| Tool | Purpose |
|------|---------|
| [sbt](https://www.scala-sbt.org/) | Build Tool |
| [Mill](https://mill-build.org/) | Build Tool |
| [Scalafix](https://scalacenter.github.io/scalafix/) | Semantic Linting & Rewrites |
| [Scalafmt](https://scalameta.org/scalafmt/) | Formatting |
| [Scalastyle](http://www.scalastyle.org/) | Style Checks |
| [Wartremover](https://www.wartremover.org/) | Compile-Time Linting |

## Related

- [scala-coding-standards](../scala-coding-standards/README.md) - General Scala coding standards and best practices
- [scala-testing](../scala-testing/README.md) - Test-writing best practices for MUnit, ScalaTest, ScalaCheck, and mocking discipline

## Usage

This skill is automatically applied when working with Scala files (`**/*.scala`) and Scala build files (`build.sbt`, `build.mill`).
