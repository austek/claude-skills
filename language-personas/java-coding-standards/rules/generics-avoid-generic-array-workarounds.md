---
title: Avoid Generic Array Creation Workarounds — Use List<T> Instead
impact: MEDIUM
impactDescription: removes unchecked casts and heap-pollution risk that generic arrays always carry
tags: [generics, arrays, type-safety, collections]
---

# Avoid Generic Array Creation Workarounds — Use List<T> Instead [MEDIUM]

## Description
Java forbids `new T[]` directly because arrays are reified (they know and enforce their element type at runtime) while generics are erased — the JVM has no `T` to check against, so a generic array could let an `ArrayStoreException` be bypassed entirely, corrupting the heap with an element of the wrong type discovered only later at an unrelated read. The common workarounds — casting `new Object[n]` to `T[]`, or using `(T[]) Array.newInstance(componentType, n)` — silence the compiler but do not remove the underlying risk; they push the unchecked cast onto whoever wrote the workaround. `List<T>` has no such gap: it is backed by an `Object[]` internally and enforces its element type entirely at the generic API boundary, with no reified array type to violate. Reach for it instead of a generic array in ordinary code; reserve actual generic array workarounds for the narrow cases — implementing a collection itself, interoperating with a legacy array-based API — where an array is unavoidable.

## Bad Example
```java
public class RingBuffer<T> {
    @SuppressWarnings("unchecked")
    private final T[] elements = (T[]) new Object[16]; // unchecked cast, heap-pollution risk

    public void set(int index, T value) {
        elements[index] = value; // no runtime check that value is actually a T
    }
}
```

## Good Example
```java
public class RingBuffer<T> {
    private final List<T> elements = new ArrayList<>(Collections.nCopies(16, null));

    public void set(int index, T value) {
        elements.set(index, value); // type-checked at the List<T> boundary, no cast anywhere
    }

    public T get(int index) {
        return elements.get(index);
    }
}
```

## Notes
- If an array is unavoidable (implementing `Collection.toArray`, or a hot numeric loop where `List<T>` boxing overhead matters), take a `Class<T>` token from the caller and build the array with `Array.newInstance(componentType, size)` rather than an unchecked cast from `Object[]`.
- `T... varargs` parameters compile to a generic array under the hood and trigger the same heap-pollution warning — annotate genuinely safe ones with `@SafeVarargs` rather than suppressing the warning at every call site.
- `List<T>.toArray(T[]::new)` (Java 11+) produces a correctly typed array from a list without any manual cast.

## References
- [Effective Java, 3rd Edition — Item 28: Prefer lists to arrays](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
