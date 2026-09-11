---
title: Keep main.rs Thin, Put Logic in lib.rs
impact: HIGH
impactDescription: Makes application logic reachable from integration tests, which binary-only code structurally cannot be
tags: [tooling, project-structure, testability, lib-main-split]
---

# Keep main.rs Thin, Put Logic in lib.rs [HIGH]

## Description
Files under `tests/` compile as separate crates that link against your library — they can reach anything `pub` in `lib.rs`, but they have no way to call into `main.rs` at all, because a binary crate isn't a dependency anything else can link against. Any real logic left sitting in `main.rs` is therefore permanently untestable from the integration layer, no matter how thorough the test suite otherwise is. Moving that logic into `lib.rs` and reducing `main.rs` to argument parsing plus a single call into the library fixes this structurally rather than by discipline — the code becomes reachable from `tests/`, and as a side effect a second binary (a `src/bin/` entry point, say) can reuse the same logic without duplicating it.

## Bad Example
```rust
// src/main.rs — all the logic lives here, unreachable by any integration test.
fn main() {
    let args = parse_args();
    let config = load_config(&args.config_path).unwrap();
    let db = connect_database(&config.db_url).unwrap();
    // Hundreds of lines of application logic follow, none of it testable
    // from tests/ since main.rs isn't a library other code can link against.
}
```

## Good Example
```rust
// src/main.rs — thin entry point
use my_app::{run, Config};

fn main() -> anyhow::Result<()> {
    let config = Config::from_env()?;
    run(config)
}

// src/lib.rs — the actual application logic, reachable from tests/
pub mod config;
pub mod database;

pub use config::Config;

pub fn run(config: Config) -> anyhow::Result<()> {
    let db = database::connect(&config.db_url)?;
    Ok(())
}
```

## Notes
- Once `run(config)` lives in `lib.rs`, `tests/integration.rs` can call `my_app::run(test_config())` directly and assert on the outcome — something structurally impossible while that logic sat in `main.rs`.
- The same split pays off again for a crate with multiple binaries: each file under `src/bin/` becomes its own thin wrapper (`Server::new()?.run()`, `Client::new()?.execute_command()`) around shared logic that lives once in `lib.rs`, instead of duplicating that logic per binary.
- `Args::parse()` (via `clap`'s derive) is a natural fit for staying in `main.rs` itself, or being defined in `lib.rs` and just invoked from `main.rs` — either way, keep the actual behavior the parsed arguments trigger inside `lib.rs`, not inline in the parsing code.
- This split costs almost nothing structurally (it's usually a handful of `pub` keywords and a `pub use`) but changes what's testable without touching a single test — it's one of the cheapest architecture decisions available for a new binary crate.

## References
- [proj-bin-dir](proj-bin-dir.md)
- [proj-mod-by-feature](proj-mod-by-feature.md)
- [test-integration-dir](../../rust-testing/rules/test-integration-dir.md)
