---
title: Configure Jackson's ObjectMapper Explicitly, Never Rely on Defaults
impact: HIGH
impactDescription: prevents silent field drops, unexpected type coercion, and version-upgrade behavior changes
tags: [reflection, serialization, jackson, json]
---

# Configure Jackson's ObjectMapper Explicitly, Never Rely on Defaults [HIGH]

## Description
Jackson's default `ObjectMapper` makes several permissive choices that are convenient for a demo and dangerous for a production API: it silently ignores unknown JSON properties instead of failing (`FAIL_ON_UNKNOWN_PROPERTIES` off by default in most Spring Boot autoconfiguration, on in raw Jackson — the actual default depends on which layer configures it, which is itself the problem), it serializes `null` fields, it doesn't register the `Optional`/`java.time` modules unless they're explicitly added, and its exact coercion rules (numeric string to number, empty string to `null`) have changed between major versions. A codebase that never configures the mapper explicitly inherits whatever Jackson's or Spring's current defaults happen to be, and a routine dependency bump can silently change how requests are parsed or responses are shaped.

Building one explicitly configured `ObjectMapper` per application (or one that layers deliberate overrides on Spring's autoconfigured instance) and reusing it — mappers are thread-safe after configuration — makes the serialization contract a decision made once and reviewed in code, not an accident of whichever library version is on the classpath.

## Bad Example
```java
public class OrderClient {
    private final ObjectMapper mapper = new ObjectMapper(); // every default is implicit and version-dependent

    public Order parse(String json) throws JsonProcessingException {
        return mapper.readValue(json, Order.class);
    }
}
```

## Good Example
```java
public final class JsonMappers {
    public static final ObjectMapper DEFAULT = JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .addModule(new Jdk8Module()) // Optional support
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, true)
            .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false)
            .serializationInclusion(JsonInclude.Include.NON_ABSENT)
            .build();
}

public class OrderClient {
    public Order parse(String json) throws JsonProcessingException {
        return JsonMappers.DEFAULT.readValue(json, Order.class);
    }
}
```

## Notes
- `FAIL_ON_UNKNOWN_PROPERTIES = true` turns a caller sending a typo'd or renamed field into an immediate, diagnosable error instead of a value that silently never arrives.
- Jackson reflectively locates a target's fields, getters, and constructors by default — the same reflective cost `reflection-serialization-avoid-field-injection` covers applies here: prefer a constructor Jackson can call directly (a canonical record constructor, or `@JsonCreator`) over reflective field injection.
- Pin module versions (`jackson-databind`, `jackson-datatype-jsr310`) together — mismatched Jackson module versions across a classpath are a common source of `NoSuchMethodError` at runtime.

## References
- [Jackson-databind Wiki — Deserialization Features](https://github.com/FasterXML/jackson-databind/wiki/Deserialization-Features)
- [Jackson-databind Wiki — Serialization Features](https://github.com/FasterXML/jackson-databind/wiki/JacksonFeatures)
