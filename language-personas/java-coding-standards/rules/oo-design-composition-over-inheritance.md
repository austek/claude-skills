---
title: Favor Composition Over Inheritance
impact: HIGH
impactDescription: changes to a reused component no longer risk silently breaking every subclass
tags: [oo-design, composition, inheritance, coupling]
---

# Favor Composition Over Inheritance [HIGH]

## Description
Inheritance couples a subclass to its superclass's implementation, not just its interface — a superclass method calling another overridable method internally can break a subclass that overrode the second method for an unrelated reason, and this coupling is invisible at the subclass's own call sites. It also only supports one axis of reuse: a class can extend only one superclass, so composing multiple independent behaviors forces them all into one inheritance chain. Composition — a class holding a reference to another and delegating to it — reuses behavior through the held object's public contract only, so the composed component's internals can change freely without breaking anything that holds it. Reserve inheritance for a genuine is-a relationship where the subclass should be substitutable everywhere the superclass is used (the Liskov substitution principle) — reach for composition by default otherwise.

## Bad Example
```java
// InstrumentedList extends ArrayList to count additions — but ArrayList's
// addAll() calls add() internally, so a call to addAll() double-counts.
public class InstrumentedList<E> extends ArrayList<E> {
    private int addCount = 0;

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    // addAll() is inherited unchanged and calls this add() per element —
    // a detail of ArrayList's implementation, not its contract.
}
```

## Good Example
```java
public class InstrumentedList<E> {
    private final List<E> delegate;
    private int addCount = 0;

    public InstrumentedList(List<E> delegate) {
        this.delegate = delegate;
    }

    public boolean add(E e) {
        addCount++;
        return delegate.add(e);
    }

    public boolean addAll(Collection<? extends E> items) {
        addCount += items.size();
        return delegate.addAll(items); // counts correctly, independent of ArrayList's internals
    }

    public int addCount() { return addCount; }
}
```

## Notes
- A class not designed and documented for safe extension should be `final`, or its extensible methods should not call each other internally in ways subclasses can't see — see `oo-design-avoid-deep-inheritance-hierarchies`.
- Composition costs a small amount of forwarding boilerplate; a well-designed interface for the composed component keeps that cost proportional to what's actually used.
- Prefer composing interfaces (dependency injection, `oo-design-dependency-injection-over-static-singletons`) over composing concrete classes, for the same substitutability benefit inheritance was meant to give.

## References
- [Effective Java, 3rd Edition — Item 18: Favor composition over inheritance](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Design Patterns (GoF) — Composition vs. Inheritance](https://en.wikipedia.org/wiki/Design_Patterns)
