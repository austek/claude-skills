---
title: Test Time-Dependent Code with a Simulated Clock
impact: HIGH
impactDescription: turns a test that waits on real seconds-to-minutes of backoff/TTL into one that runs in milliseconds, deterministically
tags: [async-testing, cats-effect, time]
---

# Test Time-Dependent Code with a Simulated Clock [HIGH]

## Description
Code that sleeps, retries with backoff, or expires a cache entry after a TTL depends on `Clock`/`IO.sleep`, and testing it against real wall-clock time makes the test both slow (it actually waits out the delay) and indirectly flaky (a loaded CI runner can shift observed timing enough to flip a boundary assertion). `cats-effect-testkit`'s `TestControl` runs an `IO` program against a simulated clock instead: `TestControl.executeEmbed(program)` executes the program to completion with every `IO.sleep` resolved instantly against virtual time, so a program that "waits" 30 seconds across three retries still runs the test in real time on the order of milliseconds, while `.timed` on the result reports the *simulated* duration for the test to assert on.

This is a strictly better default than injecting a fake `Clock` by hand for anything already using `IO.sleep`-based timing, since `TestControl` drives the same scheduling primitives the production code uses rather than requiring a parallel time abstraction threaded through the implementation purely for testability.

## Bad Example
```scala
test("retries three times with 1-second backoff") {
  val start = System.currentTimeMillis()
  retryWithBackoff(flakyCall, maxRetries = 3, delay = 1.second).unsafeRunSync()
  val elapsedMs = System.currentTimeMillis() - start
  assert(elapsedMs >= 3000) // actually waits ~3 real seconds, every run, forever
}
```

## Good Example
```scala
import cats.effect.testkit.TestControl

class RetrySuite extends munit.CatsEffectSuite:
  test("retries three times with 1-second backoff") {
    TestControl.executeEmbed {
      retryWithBackoff(flakyCall, maxRetries = 3, delay = 1.second).timed
    }.map { case (elapsed, _) =>
      assertEquals(elapsed, 3.seconds)
    }
  }
```

## Notes
- `TestControl.execute(io)` (without `Embed`) returns a handle for manually `tick`-ing virtual time and inspecting intermediate state, useful when a test needs to assert something mid-program rather than only on the final result.
- `TestControl` only simulates time for effects going through cats-effect's own scheduler (`IO.sleep`, `Temporal[F]`); a raw `Thread.sleep` inside the tested code still blocks for real (`async-testing-avoid-thread-sleep`).
- For ScalaTest-based `Future` code with injected time, the equivalent is passing an explicit `Clock`/`Scheduler` test double rather than relying on `System.currentTimeMillis()` inside production code.

## References
- [cats-effect — Testing with TestControl](https://typelevel.org/cats-effect/docs/core/test-runtime)
