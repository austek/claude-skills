---
title: Reserve Mocks for the External System Boundary
impact: MEDIUM
impactDescription: keeps mocked interaction assertions to the one layer where "was this called" is actually the question
tags: [mocking, boundaries, architecture]
---

# Reserve Mocks for the External System Boundary [MEDIUM]

## Description
Limit mock usage to traits that wrap a system this codebase doesn't own and can't reasonably fake in full — a third-party payment gateway SDK, a cloud provider's client library, a vendor webhook sender. At that boundary, the thing worth asserting often really is "did we call this with the right arguments", because there's no observable state to inspect afterward the way there is with an owned repository. Everywhere else — repositories, internal services, anything whose contract this codebase defines — an in-memory interpreter (`mocking-discipline-in-memory-interpreters`) is available and gives a stronger test, so mocking it is a downgrade, not a shortcut.

A useful test for "is this a legitimate mocking boundary": would writing a faithful in-memory fake require re-implementing the external system itself (a payment processor's fraud rules, a cloud API's eventual-consistency model)? If yes, mock the thin client wrapping it. If the trait is one this codebase owns and could fake with a `Map` or a `List` in twenty lines, write that fake instead.

## Bad Example
```scala
test("confirming an order notifies the customer") {
  val notifier = mock[CustomerNotifier] // CustomerNotifier is our own trait, easily fakeable
  OrderService(repo, notifier).confirm(orderId)
  verify(notifier).notify(customerId, "Your order is confirmed")
  // couples the test to the exact notification call shape instead of an observable outcome
}
```

## Good Example
```scala
test("confirming a payment charges the external gateway") {
  val gateway = mock[PaymentGatewayClient] // wraps a third-party SDK we don't own or control
  when(gateway.charge(any[ChargeRequest])).thenReturn(IO.pure(ChargeResult.Success("txn-1")))

  PaymentService(gateway).charge(order)

  verify(gateway).charge(ChargeRequest(order.total, order.customer.paymentMethod))
}
```

## Notes
- `CustomerNotifier` above should get an in-memory fake (a recorded list of sent notifications) that tests inspect by reading its state, the same as a repository.
- A thin wrapper around an external SDK is itself worth keeping thin, precisely so the mock at its boundary stays simple to set up and verify.
- This rule is about where mocking is *legitimate*, not mandatory — an in-memory fake for the external client is still preferable when one is feasible to write.

## References
- [Martin Fowler — Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
