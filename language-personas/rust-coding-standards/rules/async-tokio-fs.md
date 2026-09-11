---
title: Use tokio::fs Instead of std::fs in Async Code
impact: HIGH
impactDescription: Keeps the executor thread responsive during file I/O
tags: [async, filesystem, tokio, io]
---

# Use tokio::fs Instead of std::fs in Async Code [HIGH]

## Description
`std::fs` calls are blocking syscalls: the calling OS thread stops until the disk operation completes. Called from an async task, that block lands on a Tokio worker thread and prevents the scheduler from running any other task on it for the syscall's duration — the same problem as any other unyielding call inside async code. `tokio::fs` provides the same filesystem API surface (`read`, `write`, `File`, directory operations) but each call internally dispatches to `spawn_blocking`, so the calling task suspends and the worker thread stays free to run other work while the operation completes off to the side.

## Bad Example
```rust
async fn process_files(paths: &[PathBuf]) -> Result<Vec<String>> {
    let mut contents = Vec::new();
    for path in paths {
        // Blocks the entire executor thread for each file read — no
        // other task on this thread runs until the syscall returns.
        contents.push(std::fs::read_to_string(path)?);
    }
    Ok(contents)
}
```

## Good Example
```rust
use tokio::fs;

async fn process_files(paths: &[PathBuf]) -> Result<Vec<String>> {
    let mut contents = Vec::new();
    for path in paths {
        contents.push(fs::read_to_string(path).await?);
    }
    Ok(contents)
}
```

## Notes
- `std::fs` is acceptable before the async runtime starts (reading a config file in `main` prior to `Runtime::block_on`) or in a script-like `current_thread` runtime with rare, small reads, where the blocking impact is negligible.
- Each `tokio::fs` call still costs a `spawn_blocking` dispatch internally; for many small files, batch the reads with `futures::future::try_join_all` rather than looping sequential `.await`s, both for throughput and to amortize that dispatch cost.
- `tokio::fs::File` combined with `tokio::io::{AsyncReadExt, AsyncWriteExt, BufReader}` mirrors `std::io`'s buffered-reader API almost method-for-method, so porting existing synchronous I/O code is close to a drop-in swap.
- For sustained heavy I/O where even `spawn_blocking`'s dispatch overhead matters, memory-mapped files are the next lever — but that trades into `unsafe` territory and is a separate, deliberate decision, not a default.

## References
- [async-spawn-blocking](async-spawn-blocking.md)
- [async-tokio-runtime](async-tokio-runtime.md)
- [err-context-chain](err-context-chain.md)
