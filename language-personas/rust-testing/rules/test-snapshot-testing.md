---
title: Snapshot-Test Large or Structured Output with Insta
impact: MEDIUM
impactDescription: Turns brittle hand-maintained assert_eq! strings into reviewable, one-keystroke-approved diffs
tags: [testing, snapshot-testing, insta, output-verification]
---

# Snapshot-Test Large or Structured Output with Insta [MEDIUM]

## Description
Hand-writing an `assert_eq!` against a multi-line rendered error, a pretty-printed JSON payload, or generated code means maintaining a large literal string inline in the test, and updating it by hand — carefully, character for character — every time the output legitimately changes. The `insta` crate replaces that with recorded snapshot files: the first run writes the current output to a `.snap` file, every later run diffs the fresh output against that file, and `cargo insta review` shows you the diff and lets you accept or reject it with one keypress if the change was intentional. Because the `.snap` files are committed to the repository, a change to generated output shows up as a normal, reviewable line in a PR diff instead of a silent behavior change buried in test code.

## Bad Example
```rust
#[test]
fn test_config_serialization() {
    let config = Config::default();
    let json = serde_json::to_string_pretty(&config).unwrap();
    // Maintained by hand; a single formatting change breaks this test
    // for reasons unrelated to the config's actual correctness.
    assert_eq!(json, "{\n  \"timeout\": 30,\n  \"retries\": 3\n}");
}
```

## Good Example
```rust
use insta::assert_json_snapshot;

#[test]
fn test_config_serialization() {
    let config = Config::default();
    // First run: writes snapshots/test_config_serialization.snap.
    // Later runs: diffs against it and fails only on an unreviewed change.
    assert_json_snapshot!(config);
}
```

```toml
[dev-dependencies]
insta = { version = "1", features = ["json", "yaml"] }
```

## Notes
- The review loop is: run `cargo test` to generate `.snap.new` files for anything new or changed, run `cargo insta review` to see the diff and accept with `a`, then commit the resulting `.snap` file alongside the code change that caused it.
- In CI, run with `INSTA_UPDATE=no` (or the equivalent `cargo insta test --check`) so an unreviewed snapshot difference fails the build instead of silently passing or silently rewriting the committed snapshot.
- Reach for a plain `assert_eq!` on short, simple values (a boolean, a small integer, a one-word string) — snapshotting single scalars adds file-management overhead without a matching payoff; snapshots earn their keep on multi-line or structured output.
- `assert_debug_snapshot!` captures a value's `Debug` output, `assert_json_snapshot!` / `assert_yaml_snapshot!` capture serialized form, and passing a name string (`assert_debug_snapshot!("cli_help_output", output)`) keeps the snapshot filename stable even if you rename the test function later.

## References
- [test-arrange-act-assert](test-arrange-act-assert.md)
- [test-proptest-properties](test-proptest-properties.md)
- [test-doctest-examples](test-doctest-examples.md)
