---
title: Prefer a Generic Method Over a Generic Class When Only the Method Needs It
impact: LOW
impactDescription: keeps the type parameter's scope as narrow as its actual usage
tags: [generics, api-design, scope]
---

# Prefer a Generic Method Over a Generic Class When Only the Method Needs It [LOW]

## Description
A type parameter on a class applies to every field, constructor, and method in it, and forces every caller to pick a concrete type the moment they instantiate the class — even instance methods that have nothing to do with that type. When only one or two methods actually vary by type, parameterizing the whole class is wider than the need: it complicates instantiation for no benefit and can force awkward workarounds when a truly unrelated method also needs its own type parameter. A generic method (`<T> T method(...)`) scopes the type parameter to exactly the call, inferred per invocation, leaving the class itself simple to construct and reuse.

## Bad Example
```java
// Every user of Cache is forced to fix V at construction time,
// even though only get/put ever touch it — nothing else in the class needs it.
public class Cache<V> {
    private final Map<String, V> store = new ConcurrentHashMap<>();

    public V get(String key) { return store.get(key); }
    public void put(String key, V value) { store.put(key, value); }
}

Cache<Object> cache = new Cache<>(); // forced to widen, since callers store mixed types
```

## Good Example
```java
public class Cache {
    private final Map<String, Object> store = new ConcurrentHashMap<>();

    public <V> V get(String key, Class<V> type) {
        return type.cast(store.get(key));
    }

    public <V> void put(String key, V value) {
        store.put(key, value);
    }
}

Cache cache = new Cache(); // one instance, any value type per call
String name = cache.get("name", String.class);
cache.put("count", 42);
```

## Notes
- Reach for a generic class when the type parameter threads through most of the class's state and behavior — a `List<T>`, a `Repository<T, ID>` — not just an isolated method or two.
- A generic method's type parameter is usually inferred from arguments; add an explicit `Class<T>` token (as above) when nothing in the argument list pins the type down.
- Static generic methods (`static <T> T identity(T t)`) never need the enclosing class's own type parameters — keep them separate on the method itself.

## References
- [Effective Java, 3rd Edition — Item 30: Favor generic methods](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
