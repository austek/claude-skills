---
title: Never Use Thread.sleep to Wait for Async Work
impact: CRITICAL
impactDescription: replaces a guessed fixed delay — always flaky under load, always slower than necessary — with a real completion signal
tags: [async-testing, flakiness, thread-sleep]
---

# Never Use Thread.sleep to Wait for Async Work [CRITICAL]

## Description
`Thread.sleep(n)` before an assertion is a guess about how long some asynchronous operation takes, and every guess is wrong in both directions at once: on a fast, idle machine the test pays the full sleep duration even though the work finished sooner, and on a loaded CI runner the same fixed delay isn't always enough, producing an intermittent failure that has nothing to do with the code under test. Neither problem has a "right" sleep duration to fix it — any fixed number is a trade between wasted time and flakiness, not a resolution of either.

Replace it with the thing that actually knows when the async work is done: await the `Future`/`IO` directly (`async-testing-io-testrunner`) when the test controls it, poll for a condition with ScalaTest's `Eventually` trait when the completion signal isn't directly awaitable (an async callback populating a cache), or drive a simulated clock with `TestControl` (`async-testing-deterministic-clocks`) when the delay itself is the thing under test. All three replace a guessed wait with either a real completion signal or a controlled virtual clock.

## Bad Example
```scala
test("cache is populated after an async warm-up") {
  cache.warmUpAsync() // fires a background Future, returns immediately
  Thread.sleep(2000)  // guess: warm-up "usually" finishes within 2 seconds
  assert(cache.get("key").isDefined)
}
```

## Good Example
```scala
import org.scalatest.time.{Seconds, Span}

class CacheSuite extends AnyFunSuite with Eventually:
  test("cache is populated after an async warm-up") {
    cache.warmUpAsync()

    eventually(timeout(Span(2, Seconds))) {
      assert(cache.get("key").isDefined)
    }
  }
```

## Notes
- `eventually` still takes a timeout, but it polls and returns as soon as the condition holds, instead of always paying the full duration.
- If the async operation returns a `Future`/`IO` at all, prefer awaiting it directly over polling for a side effect — polling is a fallback for genuinely fire-and-forget work.
- A `Thread.sleep` used to *simulate* a slow collaborator inside a fake (not to wait for one) is a different, legitimate use; this rule is specifically about waiting on async completion.

## References
- [ScalaTest — Eventually](https://www.scalatest.org/user_guide/using_eventually)
