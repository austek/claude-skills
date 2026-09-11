---
title: Use extracting to Assert on Collection Elements' Fields
impact: MEDIUM
impactDescription: asserts the one or two fields that matter instead of requiring full-object equality
tags: [assertj, extracting, collections, readability]
---

# Use extracting to Assert on Collection Elements' Fields [MEDIUM]

## Description
`extracting(...)` projects a collection (or a single object) down to one or more fields before asserting, so a test can check "these are the SKUs, in this order" without needing every other field of `LineItem` to match via full-object equality. `assertThat(items).extracting(LineItem::sku).containsExactly("sku-1", "sku-2")` reads as a direct statement of what the test actually cares about; the alternative — building full expected `LineItem` instances just to satisfy `containsExactly(expectedItem1, expectedItem2)` — forces the test to specify fields (price, quantity, discount) that are irrelevant to what's being verified, and breaks every time an unrelated field is added to the type.

`extracting` accepts a method reference or a string property name (`extracting("sku")`, resolved via reflection) and can take multiple extractors at once, in which case the result is a stream of tuples asserted with `containsExactly(tuple("sku-1", 2), tuple("sku-2", 1))`. Use the method-reference form by default — it's compiler-checked and refactor-safe; reach for the string form only when the property isn't accessible as a method reference (a nested path AssertJ resolves reflectively).

This is the collection-focused counterpart to `mockito-argument-captor`'s reasoning: extracting the fields that matter and asserting on those directly is preferable to constructing a fully-specified expected object just to satisfy an equality check, whether the source is a captured mock argument or a collection under test.

## Bad Example
```java
@Test
void ordersContainExpectedSkus() {
    List<LineItem> expected = List.of(
        new LineItem("sku-1", Money.of("10.00"), 2, Tier.GOLD, null), // 3 fields exist only to satisfy equals()
        new LineItem("sku-2", Money.of("5.00"), 1, Tier.GOLD, null)
    );
    assertThat(order.items()).containsExactlyElementsOf(expected);
}
```

## Good Example
```java
@Test
void ordersContainExpectedSkus() {
    assertThat(order.items())
        .extracting(LineItem::sku)
        .containsExactly("sku-1", "sku-2");
}

@Test
void ordersContainExpectedSkusAndQuantities() {
    assertThat(order.items())
        .extracting(LineItem::sku, LineItem::quantity)
        .containsExactly(tuple("sku-1", 2), tuple("sku-2", 1));
}
```

## Notes
- `containsExactly` asserts both content and order; use `containsExactlyInAnyOrder` when order isn't part of the contract being tested.
- `flatExtracting` flattens a nested collection-valued extractor (e.g., extracting each order's line items across a list of orders into one flat stream).
- For a single object rather than a collection, the same `extracting` method is available directly on an object assertion: `assertThat(order).extracting(Order::status).isEqualTo(OrderStatus.PLACED)`.

## References
- [AssertJ — extracting](https://assertj.github.io/doc/#assertj-core-group-extracting-multiple-values)
