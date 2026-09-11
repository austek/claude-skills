---
title: Assert Expected Future Failures with recoverToSucceededIf
impact: MEDIUM
impactDescription: fails clearly when the future unexpectedly succeeds, instead of silently passing a test meant to check a failure
tags: [async-testing, scalatest, future]
---

# Assert Expected Future Failures with recoverToSucceededIf [MEDIUM]

## Description
In a ScalaTest `AsyncFunSuite` (or any `AsyncTestSuite`), a test body returns `Future[Assertion]`, and every combinator used to check that a `Future` fails has to stay non-blocking to keep that contract. Use `recoverToSucceededIf[ExceptionType](future)` from `org.scalatest.RecoverMethods` to assert a specific failure type, instead of falling back to `Await.result` inside a `try`/`catch`. `recoverToSucceededIf` does both halves of the check in one non-blocking call: it succeeds only if the future fails with a matching (or subtype) exception, and it explicitly fails the test if the future either succeeds or fails with the wrong exception type.

Blocking on the future with `Await.result` to run ordinary exception-handling code against it defeats the purpose of `AsyncFunSuite` — it ties up the calling thread for the future's whole duration instead of composing with the suite's execution context, the same problem `async-testing-avoid-thread-sleep` covers for polling. `recoverToSucceededIf` keeps the check entirely inside the `Future` chain.

## Bad Example
```scala
import scala.concurrent.Await
import scala.concurrent.duration._

test("rejecting a duplicate order fails") {
  Future {
    val future = OrderService(repo).confirm(duplicateOrderId)
    try
      Await.result(future, 2.seconds)
      fail("expected DuplicateOrderException")
    catch
      case _: DuplicateOrderException => succeed
  }
  // blocks a thread on Await.result instead of composing with the Future,
  // undoing the point of AsyncFunSuite's non-blocking execution model
}
```

## Good Example
```scala
import org.scalatest.RecoverMethods.recoverToSucceededIf

test("rejecting a duplicate order fails") {
  recoverToSucceededIf[DuplicateOrderException] {
    OrderService(repo).confirm(duplicateOrderId)
  }
}
```

## Notes
- `recoverToExceptionIf[ExceptionType](future)` is the sibling assertion when the test also needs to inspect the caught exception's fields, returning `Future[ExceptionType]`.
- Both helpers require an `ExecutionContext` in scope, the same as any other `Future` combinator used in the test.
- The `IO`-based equivalent is `IO.attempt`'s resulting `Either`, or MUnit's `intercept`/`interceptMessage` inside a `CatsEffectSuite` test (`async-testing-io-testrunner`).

## References
- [ScalaTest — RecoverMethods](https://www.scalatest.org/scaladoc/3.2.17/org/scalatest/RecoverMethods.html)
