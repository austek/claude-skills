---
title: Use List.copyOf/Collections.unmodifiableX at API Boundaries
impact: HIGH
impactDescription: closes the gap where a "final" collection field is still mutable
tags: [immutability, collections, api-design]
---

# Use List.copyOf/Collections.unmodifiableX at API Boundaries [HIGH]

## Description
A `final List<T> items` field only prevents reassigning `items` to a different list — it does nothing to stop `items.add(x)`. Any collection that crosses an API boundary — accepted by a constructor, returned by a getter — must be wrapped so the receiving side cannot mutate storage it does not own. `List.copyOf`, `Set.copyOf`, and `Map.copyOf` (Java 10+) do this in one call: they copy the input and return a genuinely unmodifiable view, throwing `UnsupportedOperationException` on any mutating call and rejecting `null` elements. Use `Collections.unmodifiableList`/`Set`/`Map` only when you must keep a live view over a collection that intentionally still changes elsewhere — `copyOf` is the right default because it also severs aliasing (see `immutability-defensive-copy`).

## Bad Example
```java
public class Team {
    private final List<Member> members;

    public Team(List<Member> members) {
        this.members = members; // stores the caller's own mutable list
    }

    public List<Member> getMembers() {
        return members; // returns the live internal list
    }
}

Team team = new Team(new ArrayList<>(List.of(alice, bob)));
team.getMembers().add(mallory); // caller mutates Team's internal state from outside
```

## Good Example
```java
public class Team {
    private final List<Member> members;

    public Team(List<Member> members) {
        this.members = List.copyOf(members); // copies input, immune to later caller mutation
    }

    public List<Member> getMembers() {
        return members; // already unmodifiable, safe to hand out directly
    }
}

Team team = new Team(new ArrayList<>(List.of(alice, bob)));
team.getMembers().add(mallory); // throws UnsupportedOperationException
```

## Notes
- `List.copyOf` on an already-unmodifiable `List.of(...)`/`List.copyOf(...)` result is a cheap no-op — it detects and returns the same instance instead of copying again.
- `List.copyOf` rejects `null` elements with `NullPointerException`; `Collections.unmodifiableList` does not, so switching between them is not always behavior-neutral if `null` elements were previously tolerated.
- For a field that must stay a live, growable list internally (e.g., accumulated during a builder step) but must never leak mutability, wrap only at the accessor: `return List.copyOf(internalList);` — see `immutability-builder-for-complex-construction`.

## References
- [List.copyOf (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#copyOf(java.util.Collection))
- [Collections.unmodifiableList (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html#unmodifiableList(java.util.List))
