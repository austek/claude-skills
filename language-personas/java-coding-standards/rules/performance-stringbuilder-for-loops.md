---
title: Use StringBuilder for String Concatenation Inside Loops
impact: HIGH
impactDescription: turns O(n²) string-copying into O(n) buffer growth for an n-element concatenation
tags: [performance, strings, collections-streams]
---

# Use StringBuilder for String Concatenation Inside Loops [HIGH]

## Description
`String` is immutable, so `a + b` never modifies `a` — it allocates a new `String` holding the combined characters and copies both operands into it. A single `+` outside a loop is fine; the javac compiler even desugars simple chained concatenation into an efficient `StringBuilder`/`StringConcatFactory` call automatically. But `result = result + item` written inside a loop defeats that compiler optimization, because each iteration is a separate statement: iteration *n* copies the entire string built by the previous *n-1* iterations, making the total work across an *n*-element loop quadratic rather than linear. For a loop over a handful of items the difference is noise; for a loop building a large report, log line, or SQL statement across thousands of iterations, it's the dominant cost in the method.

`StringBuilder` (unsynchronized, single-threaded use) fixes this by maintaining one mutable, amortized-growth character buffer that every `append` writes into directly, with a single `String` allocated only once at the end via `toString()`.

## Bad Example
```java
public String buildCsvLine(List<String> fields) {
    String line = "";
    for (String field : fields) {
        line = line + field + ","; // reallocates and copies the whole growing string every iteration
    }
    return line;
}
```

## Good Example
```java
public String buildCsvLine(List<String> fields) {
    StringBuilder line = new StringBuilder();
    for (String field : fields) {
        line.append(field).append(','); // appends into the existing buffer, amortized O(1) per call
    }
    return line.toString();
}

// equivalently, when the structure fits:
public String buildCsvLine(List<String> fields) {
    return String.join(",", fields);
}
```

## Notes
- `String.join`, `Collectors.joining`, and `String.format` all build their result with a `StringBuilder` (or equivalent) internally — prefer them over a hand-written loop when the shape of the data fits, for both performance and readability.
- Pre-sizing with `new StringBuilder(expectedLength)` avoids intermediate buffer-growth reallocations when the final length is known or estimable in advance.
- `StringBuffer` is the synchronized, thread-safe predecessor to `StringBuilder` — needed only when the same builder is genuinely mutated from multiple threads, which is rare enough that `StringBuilder` should be the default.

## References
- [java.lang.StringBuilder specification](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuilder.html)
- [Effective Java, 3rd Edition — Item 63: Beware the performance of string concatenation](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
