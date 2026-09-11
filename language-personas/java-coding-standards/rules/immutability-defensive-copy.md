---
title: Defensively Copy Mutable Constructor Inputs and Getter Outputs
impact: HIGH
impactDescription: closes aliasing bugs where a caller mutates state through a shared reference
tags: [immutability, defensive-copy, api-design]
---

# Defensively Copy Mutable Constructor Inputs and Getter Outputs [HIGH]

## Description
Storing a caller-supplied mutable object directly, or handing out a direct reference to internal mutable state, creates aliasing: two places hold the same reference, and a mutation through either one is visible through the other. This breaks the invariant an otherwise-immutable class is trying to guarantee. A constructor that accepts a mutable type (`Date`, `ArrayList`, a custom mutable class) must copy it before storing it; an accessor that would otherwise return a live mutable field must return a copy or an unmodifiable view instead. Records get part of this by construction (components are `final`), but a record still needs a compact constructor to copy a mutable component type — records do not defend against a mutable field's contents changing after construction.

## Bad Example
```java
public final class DateRange {
    private final Date start; // Date is mutable
    private final Date end;

    public DateRange(Date start, Date end) {
        this.start = start; // stores the caller's own Date instance
        this.end = end;
    }

    public Date getStart() {
        return start; // hands out the live internal instance
    }
}

Date start = new Date();
DateRange range = new DateRange(start, new Date());
start.setTime(0); // mutates the range's internal state from outside
range.getStart().setTime(0); // also mutates it, through the getter
```

## Good Example
```java
public final class DateRange {
    private final Instant start;
    private final Instant end;

    public DateRange(Instant start, Instant end) {
        // Instant is immutable, so no copy is needed — prefer immutable types over defending mutable ones
        this.start = Objects.requireNonNull(start, "start");
        this.end = Objects.requireNonNull(end, "end");
        if (end.isBefore(start)) {
            throw new IllegalArgumentException("end must not precede start");
        }
    }

    public Instant getStart() {
        return start; // safe: Instant cannot be mutated after creation
    }
}

// When a mutable type is unavoidable, copy on the way in and out
public final class Snapshot {
    private final byte[] payload;

    public Snapshot(byte[] payload) {
        this.payload = payload.clone(); // copy in — arrays are always mutable
    }

    public byte[] getPayload() {
        return payload.clone(); // copy out — never leak the live array
    }
}
```

## Notes
- The cheapest defense is to not accept a mutable type at all: prefer `Instant`/`LocalDate` over `Date`, and `List.copyOf` (see `immutability-unmodifiable-collections`) over storing a raw `List` reference.
- Validate the argument after copying, not before — validating the caller's original reference and then copying leaves a window (or, for concurrent callers, a race) where the copy can differ from what was checked.
- `clone()` is appropriate for arrays specifically; avoid `Object.clone()` on custom mutable classes — prefer a copy constructor or a dedicated `copy()` method instead, since `clone()`'s contract is notoriously easy to implement incorrectly.

## References
- [Effective Java, 3rd Edition — Item 50: Make defensive copies when needed](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
