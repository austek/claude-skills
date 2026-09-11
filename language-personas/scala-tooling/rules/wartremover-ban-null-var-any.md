---
title: Ban Wart.Null, Wart.Var, and Wart.Any at Compile Time
impact: HIGH
impactDescription: `null`, `var`, and untyped `Any` fail the build at the call site instead of surfacing as a runtime NPE or a silent type-safety hole
tags: [wartremover, compiler-plugin, null, var, any]
---

# Ban Wart.Null, Wart.Var, and Wart.Any at Compile Time [HIGH]

## Description
Wartremover is a compiler plugin — it runs as part of `sbt compile` itself, not a separate task — that checks each source file's AST for named anti-patterns ("warts") and reports every match as a compiler error (`wartremoverErrors`) or warning (`wartremoverWarnings`). `Wart.Null` flags any literal `null` or a type explicitly widened to admit it; `Wart.Var` flags any `var` declaration; `Wart.Any` flags any expression whose static type is exactly `Any` (a common escape hatch that erases type information the rest of the codebase relies on). All three enforce mechanically what code review can only ask for informally.

Because these are structural checks on the AST, not a config someone can misread, a `var` slipped into a pull request fails the build the same commit it's introduced, rather than surviving review because a reviewer's attention was on the logic instead of the declaration keyword.

## Bad Example
```scala
class Cache {
  var entries: Map[String, Any] = Map.empty

  def get(key: String): String =
    entries.get(key).asInstanceOf[String]
}
```

## Good Example
```scala
final class Cache(entries: Map[String, String]) {
  def get(key: String): Option[String] =
    entries.get(key)
}
```

## Notes
- Add the plugin with `libraryDependencies += compilerPlugin("org.wartremover" %% "wartremover" % "3.1.6")`, then set `Compile / compile / wartremoverErrors ++= Seq(Wart.Null, Wart.Var, Wart.Any)`.
- A specific line that genuinely needs an exempted wart (interop with a Java API returning `null`) can suppress it locally with `@SuppressWarnings(Array("org.wartremover.warts.Null"))` rather than disabling the wart project-wide.
- On an existing codebase, gate rollout with [`wartremover-scalastyle-baseline-for-legacy-code`](wartremover-scalastyle-baseline-for-legacy-code.md) instead of flipping every wart to `Errors` at once.

## References
- [Wartremover — Warts List](https://www.wartremover.org/doc/warts.html)
