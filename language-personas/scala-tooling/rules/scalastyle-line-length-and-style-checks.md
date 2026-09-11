---
title: Gate Line Length and Naming Style with scalastyle-config.xml
impact: LOW
impactDescription: catches long lines, naming violations, and magic numbers as a dedicated CI check instead of relying on reviewers to notice them
tags: [scalastyle, style, configuration, sbt-scalastyle]
---

# Gate Line Length and Naming Style with scalastyle-config.xml [LOW]

## Description
Scalastyle checks style concerns that formatting alone doesn't cover — Scalafmt reflows a line's whitespace and wrapping but never rejects a line for being too long in the first place, and neither Scalafmt nor the compiler flags a class named `myClass` instead of `MyClass`, or a bare literal like `86400` standing in for what should be a named constant. Scalastyle's `<check>` elements in `scalastyle-config.xml` cover this: `org.scalastyle.file.FileLineLengthChecker` enforces a `maxLineLength` parameter, `org.scalastyle.scalariform.ClassNamesChecker` enforces a naming regex, and `org.scalastyle.scalariform.MagicNumberChecker` flags unnamed numeric literals.

The `sbt-scalastyle` plugin adds an `scalastyle` task (and a separate `Test / scalastyle` for test sources) that reads this XML file and fails with the configured `level` (`error` fails the build, `warning` only reports) per check. Scalastyle's AST-based checks are built on Scalariform, a Scala 2 parser, so its syntax coverage lags behind Scala 3 syntax — the file-based checks (line length, file length) work regardless of dialect, but some `scalariform.*` checks may not understand newer Scala 3 constructs.

## Bad Example
```scala
// no line-length or naming check configured — a 240-character line and a lowercase
// class name both compile cleanly and pass code review unnoticed
class orderProcessor { /* ... */ }
```

## Good Example
```xml
<!-- scalastyle-config.xml -->
<scalastyle>
  <check level="error" class="org.scalastyle.file.FileLineLengthChecker" enabled="true">
    <parameters><parameter name="maxLineLength">120</parameter></parameters>
  </check>
  <check level="error" class="org.scalastyle.scalariform.ClassNamesChecker" enabled="true">
    <parameters><parameter name="regex">[A-Z][A-Za-z0-9]*</parameter></parameters>
  </check>
</scalastyle>
```

## Notes
- Run with `sbt scalastyle` (main sources) and `sbt Test / scalastyle` (test sources) — both need adding to CI explicitly.
- For an existing codebase, start every check at `level="warning"` and promote individual checks to `error` incrementally — see [`wartremover-scalastyle-baseline-for-legacy-code`](wartremover-scalastyle-baseline-for-legacy-code.md).
- Given the Scalariform limitation, teams on Scala 3 often keep only the file-based checks (line/file length) and drop AST-based naming/complexity checks in favor of Scalafix's semantic rules where equivalents exist.

## References
- [Scalastyle — Available Checks](http://www.scalastyle.org/rules-1.0.0.html)
