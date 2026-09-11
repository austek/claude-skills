---
title: Never Restate a Parameter's Type or Name in Its Own Description
impact: LOW
impactDescription: keeps every line of a doc comment carrying information the signature doesn't already show
tags: [scaladoc, documentation, comments]
---

# Never Restate a Parameter's Type or Name in Its Own Description [LOW]

## Description
An `@param id a String representing the user id` line on a parameter already declared as `id: UserId` in the signature transfers nothing a reader didn't already have — the signature already says the parameter is called `id` and already says its type (arguably more precisely than the prose does, since "a String" is even wrong once the parameter is a `UserId` and not a raw `String`). This is signature narration: prose that describes the shape the compiler already enforces and the reader can already see, rather than the one thing a doc comment can add that the signature can't — meaning. `@return an Option of User, which is Some if found and None if not` has the same problem in the other direction: `Option[User]` already says exactly that at the type level; restating it in English adds a second, unenforced copy of the same fact that can drift out of sync with the type the moment either one changes.

Write `@param`/`@return` prose only when there's a fact the signature genuinely can't express — a unit, a valid range, what triggers each distinct outcome, a side effect. Where a parameter or return type is fully self-explanatory once named well, skip the tag entirely rather than filling it with a restatement; a doc comment with fewer tags that each say something new reads faster and stays trustworthy longer than one with a tag on every parameter regardless of whether it has anything to add.

## Bad Example
```scala
/** Finds a user.
  *
  * @param id a String representing the user id
  * @return an Option of User, which is Some if found and None if not
  */
def findUser(id: UserId): Option[User] = ???
```

## Good Example
```scala
/** Looks up a user by id, without hitting the database if a cached entry exists. */
def findUser(id: UserId): Option[User] = ???
```

## Notes
- This mirrors the workspace-wide doc-comment rule — "state the contract only, no signature narration" — applied specifically to Scaladoc's `@param`/`@return`/`@tparam` tags.
- `scaladoc-public-api-contract` covers the complementary failure mode — a doc comment that says nothing beyond the method name; this rule covers one that says nothing beyond the parameter list.
- A parameter genuinely worth a tag is one where the type alone leaves something unstated: a `timeout: FiniteDuration` parameter's *unit* is in the type already, but whether zero means "no timeout" or "fail immediately" is not, and belongs in the tag.
- Renaming a poorly-named parameter to something self-explanatory is very often a better fix than writing a tag to compensate for the name.

## References
- [Scaladoc for Library Authors](https://docs.scala-lang.org/style/scaladoc.html)
