---
title: Choose Intention-Revealing Names
impact: HIGH
impactDescription: cuts the time a reader spends reconstructing intent from the code body
tags: [naming, readability, api-design]
---

# Choose Intention-Revealing Names [HIGH]

## Description
A name is the cheapest documentation available and the one every reader sees first — it should answer why a variable, method, or class exists and what it holds or does, without forcing the reader into the implementation. A name like `d` or `list` or `process()` tells the reader nothing they didn't already know from the type; `elapsedTimeInDays` or `activeCustomers` or `cancelExpiredSubscriptions()` tells them what the code means. This matters most for anything with a lifetime beyond the line it's declared on — fields, parameters, public methods, and class names all get read far more often than they get written, so a few extra characters chosen well pay for themselves on every future read. A short-lived loop index or lambda parameter in a two-line scope is the one place brevity still wins, because the whole scope is visible at a glance.

Vague names also hide missing abstractions — a method called `handle(Object o)` usually means the real operation was never named, and naming it well often exposes that the method is doing two unrelated things.

## Bad Example
```java
public class Mgr {
    private List<Object> l = new ArrayList<>();

    public void proc(Object o, int f) {
        if (f == 1) {
            l.add(o);
        }
    }
}
```

## Good Example
```java
public class SubscriptionRenewalQueue {
    private final List<Subscription> pendingRenewals = new ArrayList<>();

    public void enqueueIfEligible(Subscription subscription, RenewalStatus status) {
        if (status == RenewalStatus.ELIGIBLE) {
            pendingRenewals.add(subscription);
        }
    }
}
```

## Notes
- Prefer a domain term the business already uses (`invoice`, `tenant`) over a generic one (`entity`, `item`) — it lets code and conversation share vocabulary.
- A name that needs a trailing comment to explain it (`int f; // 1 = eligible`) is a sign the name, or the type, is wrong — an enum for `f` removes the comment entirely.
- Renaming is a mechanical, low-risk refactor with IDE support — there is rarely a reason to keep a name that stopped fitting once code around it changed.

## References
- [Effective Java, 3rd Edition — Item 68: Adhere to generally accepted naming conventions](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Clean Code — Chapter 2: Meaningful Names](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
