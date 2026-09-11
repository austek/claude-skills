---
title: Scope Checkstyle to Structural Checks, Not Formatting
impact: MEDIUM
impactDescription: removes the two-source-of-truth conflict between a linter and a formatter fighting over whitespace
tags: [checkstyle, lint, static-analysis, code-style]
---

# Scope Checkstyle to Structural Checks, Not Formatting [MEDIUM]

## Description
Checkstyle is a source-based linter driven by an XML rule set — typically started from `sun_checks.xml` or `google_checks.xml` and customized — that can check both structural properties (cyclomatic complexity limits, method length, magic-number literals, javadoc presence on public API, naming conventions) and purely mechanical formatting (brace placement, whitespace, line length). The structural checks require a judgment call about the code that no formatter can make automatically; the formatting checks are exactly what a formatter like Spotless/google-java-format already enforces by rewriting the file.

Running both in the same build without deconflicting them creates two sources of truth that can disagree: Checkstyle fails a build on a brace-placement violation while the formatter, run separately, would have silently fixed that same line to a different style. The fix is to remove Checkstyle's formatting-only modules (`LeftCurly`, `WhitespaceAround`, and similar) from the ruleset entirely and let the formatter own 100% of mechanical style, leaving Checkstyle to gate on things a formatter structurally cannot fix — a method that's grown to 40 lines, a cyclomatic complexity of 15, a magic number that should be a named constant.

Severity levels (`ignore`/`info`/`warning`/`error`) on each check let a team ratchet up strictness incrementally, the same pattern Error Prone uses for promoting individual checks — start new structural checks at `warning` to surface findings without breaking the build, then promote to `error` once the existing codebase is clean for that check.

## Bad Example
```xml
<module name="LeftCurly"/> <!-- also enforced, differently, by the formatter -->
<module name="WhitespaceAround"/> <!-- also enforced, differently, by the formatter -->
```
A file that satisfies the formatter can still fail Checkstyle, or vice versa, depending on which ran last.

## Good Example
```xml
<module name="CyclomaticComplexity">
    <property name="max" value="10"/>
</module>
<module name="MethodLength">
    <property name="max" value="60"/>
</module>
<module name="MagicNumber"/>
```

## Notes
- A `<suppressions>` file handles justified per-file or per-line exceptions without weakening the check globally — prefer it over lowering a check's severity repo-wide for one legitimate outlier.
- Checkstyle's `MagicNumberCheck` mechanically enforces `../../java-coding-standards/rules/anti-patterns-no-magic-numbers.md` — wiring it in turns that rule from a review comment into a CI gate.
- Running Checkstyle on source rather than bytecode means it can check things SpotBugs (`static-analysis-spotbugs-ci-gate.md`) cannot, such as javadoc presence, but it also can't see cross-method or cross-class bytecode-level issues SpotBugs catches.

## References
- [Checkstyle](https://checkstyle.org/)
- [Checkstyle — Available Checks](https://checkstyle.org/checks.html)
