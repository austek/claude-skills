---
title: Wrap Frequent Reads/Writes in BufReader/BufWriter
impact: HIGH
impactDescription: Cuts syscall count by orders of magnitude on line-by-line or record-by-record IO
tags: [performance, io, buffering, syscalls]
---

# Wrap Frequent Reads/Writes in BufReader/BufWriter [HIGH]

## Description
A `File` or socket doesn't know how much data a caller is about to ask for — it treats `read()`/`write()` as a full round trip through the kernel each time, paying a fixed context-switch cost whether the call moves one byte or sixty-four thousand. Code structured around single-byte or single-record calls therefore pays that fixed overhead once per byte or record instead of once per useful chunk of work, and on a file of any real size that turns a handful of cheap operations into millions of expensive ones. `BufReader`/`BufWriter` fix this without touching the calling code's logic at all: they sit between the caller and the file descriptor holding an in-memory buffer (8 KiB by default), so many small logical reads or writes get serviced from — or accumulated into — that buffer and only occasionally trigger an actual kernel call.

## Bad Example
```rust
use std::fs::File;
use std::io::{Read, Write};

fn count_lines(path: &str) -> std::io::Result<usize> {
    let mut file = File::open(path)?;
    let mut count = 0;
    let mut byte = [0u8; 1];
    while file.read(&mut byte)? > 0 { // one syscall per byte
        if byte[0] == b'\n' { count += 1; }
    }
    Ok(count)
}
```

## Good Example
```rust
use std::fs::File;
use std::io::{self, BufRead, BufReader, BufWriter, Write};

fn count_lines(path: &str) -> io::Result<usize> {
    let reader = BufReader::new(File::open(path)?);
    Ok(reader.lines().count()) // reads happen in large batched chunks
}

fn write_records(path: &str, records: &[String]) -> io::Result<()> {
    let mut writer = BufWriter::new(File::create(path)?);
    for record in records {
        writer.write_all(record.as_bytes())?;
        writer.write_all(b"\n")?;
    }
    writer.flush() // required: drop() discards a failed flush silently
}
```

## Notes
- `BufWriter` must be flushed explicitly — its `Drop` impl tries to flush but has nowhere to report an error if that fails, so a write failure right at the end of a program can vanish unless `.flush()` is called before the writer goes out of scope.
- The default 8 KiB buffer is a reasonable general default; sequential reads of a genuinely large file can benefit further from `BufReader::with_capacity(64 * 1024, file)` or larger, which reduces syscall count even further at the cost of more memory held per open file.
- Network sockets (`TcpStream` and similar) benefit identically — an unbuffered `write` on a socket can turn into a tiny TCP segment per call, and wrapping in `BufWriter` batches those the same way it batches file writes. Don't double-wrap a stream that's already internally buffered (some async IO types are).

## References
- [mem-with-capacity](mem-with-capacity.md)
- [perf-profile-first](perf-profile-first.md)
