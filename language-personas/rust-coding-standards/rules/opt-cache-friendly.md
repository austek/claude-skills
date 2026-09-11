---
title: Organize Data for Cache-Efficient Access Patterns
impact: HIGH
impactDescription: Can produce order-of-magnitude gains — an L3 miss costs ~100+ cycles vs ~4 for L1
tags: [optimization, performance, cache, data-layout]
---

# Organize Data for Cache-Efficient Access Patterns [HIGH]

## Description
Cache misses dominate the cost of many hot loops far more than the arithmetic they contain. An array-of-structs (AoS) layout forces the CPU to load an entire struct — including fields the loop never touches — just to reach the few bytes it needs, wasting cache-line capacity. A struct-of-arrays (SoA) layout instead stores each field in its own contiguous array, so a loop that only touches `position` and `velocity` streams through memory it actually uses. The same principle applies to splitting rarely-accessed ("cold") fields out of a hot struct, and to avoiding pointer-chasing structures like linked lists in favor of contiguous vectors.

## Bad Example
```rust
// Array of Structs (AoS): touching one field still loads the whole struct
struct Particle {
    position: [f32; 3],
    velocity: [f32; 3],
    mass: f32,
    id: u64,
}

fn update_positions(particles: &mut [Particle], dt: f32) {
    for p in particles {
        // Loads all 32+ bytes of Particle per iteration to reach 24 useful bytes
        for i in 0..3 {
            p.position[i] += p.velocity[i] * dt;
        }
    }
}
```

## Good Example
```rust
// Struct of Arrays (SoA): each field is its own contiguous, hot array
struct Particles {
    positions: Vec<[f32; 3]>,
    velocities: Vec<[f32; 3]>,
    // rarely-touched fields live in a separate Cold struct, not interleaved here
}

fn update_positions(p: &mut Particles, dt: f32) {
    for (pos, vel) in p.positions.iter_mut().zip(&p.velocities) {
        for i in 0..3 {
            pos[i] += vel[i] * dt;
        }
    }
}
```

## Notes
- Hot/cold splitting: pull rarely-accessed fields (names, timestamps, metadata maps) into a separate struct so the hot loop's working set stays small.
- `#[repr(C, align(64))]` pins a type to a cache line — useful both for deliberate alignment and to pad atomics and prevent false sharing between threads.
- Measure with `perf stat -e cache-references,cache-misses` (Linux) or Cachegrind before restructuring; AoS is often fine when the loop touches most fields anyway.
- Prefer contiguous vectors over linked structures (`Box`-chained nodes) for anything iterated in bulk — each `Box` hop is a likely cache miss.

## References
- [opt-bounds-check](opt-bounds-check.md)
