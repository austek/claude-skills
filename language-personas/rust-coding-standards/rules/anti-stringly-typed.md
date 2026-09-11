---
title: Watch for Two &str Parameters That Can Be Silently Swapped
impact: MEDIUM
impactDescription: A call-site argument swap that the compiler has no way to catch
tags: [anti-pattern, type-safety, api-design, code-smell]
---

# Watch for Two &str Parameters That Can Be Silently Swapped [MEDIUM]

## Description
Beyond the general risk of a `&str` parameter accepting an unvalidated typo (covered by `type-no-stringly`), a function taking two or more `&str` arguments with related but different meanings creates a second, sharper failure mode: nothing stops a caller from passing them in the wrong order. `process_order(status, priority)` and `process_order(priority, status)` both type-check identically when both parameters are `&str`, so a transposition typo compiles clean and only surfaces as wrong behavior at runtime — potentially only under a specific input that exercises the mismatch. Distinct enum or newtype parameters close this off structurally: `process_order(OrderStatus, Priority)` makes the swapped call a type error the compiler catches before the code ever runs.

## Bad Example
```rust
fn process_order(status: &str, priority: &str) { /* ... */ }

process_order("high", "pending"); // arguments swapped -- compiles fine, wrong at runtime
```

## Good Example
```rust
enum OrderStatus { Pending, Processing, Completed, Cancelled }
enum Priority { Low, Medium, High, Critical }

fn process_order(status: OrderStatus, priority: Priority) { /* ... */ }

// process_order(Priority::High, OrderStatus::Pending); // compile error: types don't match
process_order(OrderStatus::Pending, Priority::High); // only the correct order compiles
```

## Notes
- The risk scales with how similar the two `&str` values look at a glance — two parameters both drawn from short, generic-sounding word lists (`"high"`/`"pending"`, `"red"`/`"active"`) are exactly the case where a reviewer's eye also glosses over a swap.
- This is a narrower, call-site-focused case of the broader anti-pattern in `type-no-stringly` — that entry covers the general fix (enums, validated newtypes, parsing at the boundary); this one is specifically about recognizing multi-`&str`-parameter functions as an argument-order risk during review.
- A function reduced to a single `&str` parameter doesn't have this particular failure mode (there's nothing to swap it with) — the risk is specific to *multiple* same-typed string parameters sitting next to each other in a signature.

## References
- [type-no-stringly](type-no-stringly.md)
- [api-newtype-safety](api-newtype-safety.md)
- [type-newtype-validated](type-newtype-validated.md)
