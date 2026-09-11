---
name: rust-coding-standards
description: Rust coding standards and best practices covering ownership and borrowing, error handling, async programming, API design, unsafe code, and performance. Use when writing, reviewing, or refactoring Rust code, or when the user asks about Rust style, design, or best practices.
paths:
  - "**/*.rs"
  - "Cargo.toml"
---

# Rust Coding Standards

A comprehensive collection of Rust coding standards and best practices, adapted from the leonardomso/rust-skills catalog (MIT licensed — see [NOTICE.md](NOTICE.md)) for AI agents and LLMs to generate idiomatic, safe, and performant Rust code.

## Categories

### Async Programming [HIGH]
Correct, non-blocking use of async/await, the Tokio runtime, channels, and cancellation.

| Rule | Description |
|------|-------------|
| [async-async-fn-bounds](rules/async-async-fn-bounds.md) | Use AsyncFn bounds instead of F: Fn() -> Fut |
| [async-bounded-channel](rules/async-bounded-channel.md) | Use bounded channels to apply backpressure |
| [async-broadcast-pubsub](rules/async-broadcast-pubsub.md) | Use broadcast for Pub/Sub where all subscribers see all messages |
| [async-cancel-safety](rules/async-cancel-safety.md) | Ensure futures in select! branches are cancellation-safe |
| [async-cancellation-token](rules/async-cancellation-token.md) | Use CancellationToken for graceful shutdown |
| [async-clone-before-await](rules/async-clone-before-await.md) | Clone Arc/Rc data before await points |
| [async-fn-in-trait](rules/async-fn-in-trait.md) | Prefer native async fn in traits over async-trait |
| [async-join-parallel](rules/async-join-parallel.md) | Use join! to run independent futures concurrently |
| [async-joinset-structured](rules/async-joinset-structured.md) | Use JoinSet for dynamic collections of spawned tasks |
| [async-mpsc-queue](rules/async-mpsc-queue.md) | Use tokio::sync::mpsc for task-to-task message queues |
| [async-no-lock-await](rules/async-no-lock-await.md) | Never hold a Mutex/RwLock guard across an await point |
| [async-oneshot-response](rules/async-oneshot-response.md) | Use oneshot for single request-response replies |
| [async-select-racing](rules/async-select-racing.md) | Use select! to race futures and cancel the losers |
| [async-spawn-blocking](rules/async-spawn-blocking.md) | Use spawn_blocking for CPU-intensive or blocking work |
| [async-tokio-fs](rules/async-tokio-fs.md) | Use tokio::fs instead of std::fs in async code |
| [async-tokio-runtime](rules/async-tokio-runtime.md) | Configure the Tokio runtime for the actual workload |
| [async-try-join](rules/async-try-join.md) | Use try_join! for concurrent fallible operations |
| [async-watch-latest](rules/async-watch-latest.md) | Use watch for broadcasting the latest value |

### Type Safety [HIGH]
Encode invariants in the type system so invalid states become unrepresentable.

| Rule | Description |
|------|-------------|
| [type-deref-coercion](rules/type-deref-coercion.md) | Implement Deref only for smart pointers and transparent wrappers |
| [type-display-vs-debug](rules/type-display-vs-debug.md) | Use Display for users and Debug for diagnostics — never swap them |
| [type-enum-states](rules/type-enum-states.md) | Use enums for mutually exclusive states |
| [type-generic-bounds](rules/type-generic-bounds.md) | Place trait bounds where needed, prefer where clauses |
| [type-never-diverge](rules/type-never-diverge.md) | Use the never type for functions that never return |
| [type-newtype-ids](rules/type-newtype-ids.md) | Wrap IDs in newtypes instead of raw integers |
| [type-newtype-validated](rules/type-newtype-validated.md) | Use newtypes to enforce validation at construction |
| [type-no-stringly](rules/type-no-stringly.md) | Avoid stringly-typed APIs |
| [type-numeric-fmt](rules/type-numeric-fmt.md) | Implement Hex/Octal/Binary formatting for numeric newtypes |
| [type-option-nullable](rules/type-option-nullable.md) | Use Option for values that might not exist |
| [type-phantom-marker](rules/type-phantom-marker.md) | Use PhantomData to express type relationships at zero cost |
| [type-repr-transparent](rules/type-repr-transparent.md) | Use #[repr(transparent)] for FFI newtypes |
| [type-result-fallible](rules/type-result-fallible.md) | Use Result for operations that can fail |

### Serialization (Serde) [HIGH]
Serde patterns for robust, forward- and backward-compatible wire formats.

| Rule | Description |
|------|-------------|
| [serde-custom-with](rules/serde-custom-with.md) | Customize field (De)serialization with serde with modules |
| [serde-default-compat](rules/serde-default-compat.md) | Use serde default for optional and backward-compatible fields |
| [serde-deny-unknown-fields](rules/serde-deny-unknown-fields.md) | Reject unexpected keys with serde deny_unknown_fields |
| [serde-enum-representation](rules/serde-enum-representation.md) | Choose Serde enum tagging deliberately |
| [serde-flatten](rules/serde-flatten.md) | Inline shared fields with serde flatten |
| [serde-rename-all](rules/serde-rename-all.md) | Match external naming with serde rename_all |
| [serde-skip-empty](rules/serde-skip-empty.md) | Omit empty fields with serde skip_serializing_if |
| [serde-try-from-validate](rules/serde-try-from-validate.md) | Validate during deserialization with serde try_from |

### Observability [HIGH]
Structured logging, tracing, and metrics for diagnosing production behavior.

| Rule | Description |
|------|-------------|
| [obs-error-chain](rules/obs-error-chain.md) | Log the full error chain once, at the layer that handles it |
| [obs-instrument-spans](rules/obs-instrument-spans.md) | Attach async work to spans with #[instrument], never a held guard |
| [obs-levels-filter](rules/obs-levels-filter.md) | Pick log levels by urgency and filter at runtime with EnvFilter |
| [obs-library-facade](rules/obs-library-facade.md) | Libraries emit through tracing/log, only binaries install a subscriber |
| [obs-no-sensitive-data](rules/obs-no-sensitive-data.md) | Never let secrets or PII reach a log line or span field |
| [obs-structured-fields](rules/obs-structured-fields.md) | Record values as structured fields, not interpolated into the message |
| [obs-tracing-over-log](rules/obs-tracing-over-log.md) | Reach for tracing instead of println! or bare log for diagnostics |

### Unsafe Rust [HIGH]
Minimize, document, and verify the unsafe code that upholds Rust's safety guarantees.

| Rule | Description |
|------|-------------|
| [unsafe-extern-block](rules/unsafe-extern-block.md) | Wrap extern blocks in unsafe extern with per-item safety |
| [unsafe-maybeuninit](rules/unsafe-maybeuninit.md) | Use MaybeUninit for uninitialized memory |
| [unsafe-minimize-scope](rules/unsafe-minimize-scope.md) | Keep unsafe blocks as small as possible |
| [unsafe-miri-ci](rules/unsafe-miri-ci.md) | Run cargo miri test in CI for every unsafe-containing crate |
| [unsafe-no-mangle-unsafe](rules/unsafe-no-mangle-unsafe.md) | Write #[unsafe(no_mangle)] instead of bare attribute forms |
| [unsafe-safety-comment](rules/unsafe-safety-comment.md) | Document every unsafe block with a SAFETY comment |
| [unsafe-send-sync-manual](rules/unsafe-send-sync-manual.md) | Document invariants when manually implementing Send or Sync |

### Numeric Types [HIGH]
Overflow-safe arithmetic and correct numeric conversions.

| Rule | Description |
|------|-------------|
| [num-cast-try-from](rules/num-cast-try-from.md) | Avoid as for narrowing casts — use TryFrom |
| [num-float-compare](rules/num-float-compare.md) | Never compare floats with == — use tolerance or total_cmp |
| [num-nonzero](rules/num-nonzero.md) | Use NonZero types to forbid zero at the type level |
| [num-overflow-explicit](rules/num-overflow-explicit.md) | Handle integer overflow explicitly |
| [num-saturating-clamp](rules/num-saturating-clamp.md) | Bound values with clamp and saturating arithmetic |

### Concurrency [HIGH]
Safe, efficient shared-state concurrency outside of async.

| Rule | Description |
|------|-------------|
| [conc-atomic-ordering](rules/conc-atomic-ordering.md) | Use the weakest correct memory ordering for atomics |
| [conc-rayon-par-iter](rules/conc-rayon-par-iter.md) | Use rayon's par_iter for CPU-bound data parallelism |
| [conc-scoped-threads](rules/conc-scoped-threads.md) | Use thread::scope to borrow stack data across threads |
| [conc-thread-local](rules/conc-thread-local.md) | Prefer thread_local! over static mut |

### API Design [MEDIUM]
Public interface conventions that make crates predictable and hard to misuse.

| Rule | Description |
|------|-------------|
| [api-builder-must-use](rules/api-builder-must-use.md) | Mark builder methods with #[must_use] |
| [api-builder-pattern](rules/api-builder-pattern.md) | Use the builder pattern for complex construction |
| [api-common-traits](rules/api-common-traits.md) | Implement common traits for public types |
| [api-default-impl](rules/api-default-impl.md) | Implement Default for types with sensible defaults |
| [api-extension-trait](rules/api-extension-trait.md) | Use extension traits to add methods to external types |
| [api-from-not-into](rules/api-from-not-into.md) | Implement from, not into |
| [api-impl-asref](rules/api-impl-asref.md) | Use AsRef<T> when you only need to borrow |
| [api-impl-fromiterator](rules/api-impl-fromiterator.md) | Implement FromIterator, Extend, and IntoIterator for collection types |
| [api-impl-into](rules/api-impl-into.md) | Accept impl Into<T> for flexible APIs |
| [api-must-use](rules/api-must-use.md) | Mark types and functions #[must_use] when ignoring them is a bug |
| [api-newtype-safety](rules/api-newtype-safety.md) | Use newtypes to prevent mixing semantically different values |
| [api-non-exhaustive](rules/api-non-exhaustive.md) | Use #[non_exhaustive] for forward-compatible public types |
| [api-operator-overload](rules/api-operator-overload.md) | Overload operators only when semantics are obvious |
| [api-parse-dont-validate](rules/api-parse-dont-validate.md) | Parse into validated types at boundaries |
| [api-sealed-trait](rules/api-sealed-trait.md) | Use sealed traits to prevent external implementations |
| [api-serde-optional](rules/api-serde-optional.md) | Make Serde a feature flag, not a hard dependency |
| [api-typestate](rules/api-typestate.md) | Use the typestate pattern to encode state machine invariants |

### Memory Management [MEDIUM]
Allocation, layout, and lifetime patterns that cut memory pressure and copies.

| Rule | Description |
|------|-------------|
| [mem-arena-allocator](rules/mem-arena-allocator.md) | Use arena allocators for batch allocations |
| [mem-arrayvec](rules/mem-arrayvec.md) | Use ArrayVec for fixed-capacity stack collections |
| [mem-assert-type-size](rules/mem-assert-type-size.md) | Assert type sizes to guard against silent bloat |
| [mem-avoid-format](rules/mem-avoid-format.md) | Avoid format! when a String literal works |
| [mem-box-large-variant](rules/mem-box-large-variant.md) | Box large enum variants to shrink the whole enum |
| [mem-boxed-slice](rules/mem-boxed-slice.md) | Use Box<[T]> for fixed-size heap data |
| [mem-clone-from](rules/mem-clone-from.md) | Use clone_from to reuse allocations when repeatedly cloning |
| [mem-compact-string](rules/mem-compact-string.md) | Use compact String types for memory-constrained storage |
| [mem-drop-order](rules/mem-drop-order.md) | Know and control Drop order |
| [mem-reuse-collections](rules/mem-reuse-collections.md) | Clear and reuse collections instead of recreating them in loops |
| [mem-smaller-integers](rules/mem-smaller-integers.md) | Use appropriately-sized integers to reduce memory footprint |
| [mem-smallvec](rules/mem-smallvec.md) | Use SmallVec for usually-small collections |
| [mem-take-replace](rules/mem-take-replace.md) | Use mem::take / mem::replace to move out of &mut without cloning |
| [mem-thinvec](rules/mem-thinvec.md) | Use ThinVec for nullable collections with minimal overhead |
| [mem-with-capacity](rules/mem-with-capacity.md) | Pre-allocate with with_capacity when size is known |
| [mem-write-over-format](rules/mem-write-over-format.md) | Use write! into existing buffers instead of format! allocations |
| [mem-zero-copy](rules/mem-zero-copy.md) | Use zero-copy patterns with slices and bytes |

### Naming [MEDIUM]
Naming conventions that keep Rust APIs idiomatic and self-documenting.

| Rule | Description |
|------|-------------|
| [name-acronym-word](rules/name-acronym-word.md) | Capitalize acronyms as ordinary words |
| [name-as-free](rules/name-as-free.md) | Reserve the as_ prefix for free reference conversions |
| [name-consts-screaming](rules/name-consts-screaming.md) | Name constants and statics in SCREAMING_SNAKE_CASE |
| [name-crate-no-rs](rules/name-crate-no-rs.md) | Skip the -rs / -rust suffix on crate names |
| [name-funcs-snake](rules/name-funcs-snake.md) | Name functions, methods, and variables in snake_case |
| [name-into-ownership](rules/name-into-ownership.md) | Use into_ for conversions that consume self |
| [name-is-has-bool](rules/name-is-has-bool.md) | Prefix boolean methods with is_/has_/can_ |
| [name-iter-convention](rules/name-iter-convention.md) | Offer iter/iter_mut/into_iter as a matched set |
| [name-iter-method](rules/name-iter-method.md) | Keep Iterator-producing methods to a fixed naming vocabulary |
| [name-iter-type-match](rules/name-iter-type-match.md) | Name Iterator types after the method that produces them |
| [name-lifetime-short](rules/name-lifetime-short.md) | Keep lifetime names short and conventional |
| [name-no-get-prefix](rules/name-no-get-prefix.md) | Drop the get_ prefix on plain field accessors |
| [name-to-expensive](rules/name-to-expensive.md) | Reserve the to_ prefix for conversions that cost something |
| [name-type-param-single](rules/name-type-param-single.md) | Name generic type parameters with a single uppercase letter |
| [name-types-camel](rules/name-types-camel.md) | Name types, traits, and enums in UpperCamelCase |
| [name-variants-camel](rules/name-variants-camel.md) | Name enum variants in UpperCamelCase |

### Anti-Patterns [MEDIUM]
Recognize and avoid common mistakes that compile cleanly but degrade correctness or performance.

| Rule | Description |
|------|-------------|
| [anti-clone-excessive](rules/anti-clone-excessive.md) | Recognize cloning where a borrow would have worked |
| [anti-collect-intermediate](rules/anti-collect-intermediate.md) | Recognize the collect-then-iterate-again anti-pattern |
| [anti-empty-catch](rules/anti-empty-catch.md) | Never let an error vanish into an empty match arm |
| [anti-expect-lazy](rules/anti-expect-lazy.md) | Don't reach for expect() just because it has a message |
| [anti-format-hot-path](rules/anti-format-hot-path.md) | Notice format! allocating inside a loop |
| [anti-index-over-iter](rules/anti-index-over-iter.md) | Notice a for i in 0..len loop where an Iterator was available |
| [anti-lock-across-await](rules/anti-lock-across-await.md) | Treat a lock guard held across .await as an immediate red flag |
| [anti-over-abstraction](rules/anti-over-abstraction.md) | Don't generalize before a second concrete use case exists |
| [anti-panic-expected](rules/anti-panic-expected.md) | Don't let panic! stand in for an ordinary failure path |
| [anti-premature-optimize](rules/anti-premature-optimize.md) | Don't add optimization complexity before a profiler asks for it |
| [anti-string-for-str](rules/anti-string-for-str.md) | Don't narrow a function parameter to &string |
| [anti-stringly-typed](rules/anti-stringly-typed.md) | Watch for two &str parameters that can be silently swapped |
| [anti-type-erasure](rules/anti-type-erasure.md) | Reach for Box<dyn Trait> only when types genuinely vary at runtime |
| [anti-unwrap-abuse](rules/anti-unwrap-abuse.md) | Treat .unwrap() in non-test code as a question, not an answer |
| [anti-vec-for-slice](rules/anti-vec-for-slice.md) | Don't narrow a function parameter to &Vec<T> |

### Performance [MEDIUM]
General optimization patterns for CPU- and allocation-bound code.

| Rule | Description |
|------|-------------|
| [perf-ahash](rules/perf-ahash.md) | Swap SipHash for ahash/FxHash when keys aren't attacker-controlled |
| [perf-black-box-bench](rules/perf-black-box-bench.md) | Wrap benchmark inputs and outputs in black_box |
| [perf-chain-avoid](rules/perf-chain-avoid.md) | Avoid Iterator::chain inside a hot inner loop |
| [perf-collect-into](rules/perf-collect-into.md) | Reuse a collection's allocation instead of collecting fresh each time |
| [perf-collect-once](rules/perf-collect-once.md) | Chain Iterator adapters and collect exactly once |
| [perf-drain-reuse](rules/perf-drain-reuse.md) | Use drain() to move elements out without losing the allocation |
| [perf-entry-api](rules/perf-entry-api.md) | Use the entry API instead of check-then-insert on a map |
| [perf-extend-batch](rules/perf-extend-batch.md) | Batch insertions with extend() instead of repeated push() |
| [perf-io-buffering](rules/perf-io-buffering.md) | Wrap frequent Reads/Writes in BufReader/BufWriter |
| [perf-iter-lazy](rules/perf-iter-lazy.md) | Keep an Iterator pipeline lazy until the final consumer |
| [perf-iter-over-index](rules/perf-iter-over-index.md) | Traverse slices with iterators, not manual index loops |
| [perf-profile-first](rules/perf-profile-first.md) | Profile before optimizing — don't guess where the time goes |
| [perf-release-profile](rules/perf-release-profile.md) | Tune [profile.release] instead of shipping cargo's defaults |

### Error Handling [MEDIUM]
Fallible-value patterns that keep errors typed, propagated, and actionable.

| Rule | Description |
|------|-------------|
| [err-anyhow-app](rules/err-anyhow-app.md) | Use Anyhow for application error handling |
| [err-context-chain](rules/err-context-chain.md) | Add context to errors with context() and with_context() |
| [err-custom-type](rules/err-custom-type.md) | Define custom error types for domain-specific failures |
| [err-doc-errors](rules/err-doc-errors.md) | Document error conditions with an errors section |
| [err-expect-bugs-only](rules/err-expect-bugs-only.md) | Reserve expect() for invariants, not user or external errors |
| [err-from-impl](rules/err-from-impl.md) | Implement from for error conversions to enable the question-mark operator |
| [err-lowercase-msg](rules/err-lowercase-msg.md) | Start error messages lowercase, no trailing punctuation |
| [err-no-unwrap-prod](rules/err-no-unwrap-prod.md) | Avoid unwrap() in production code |
| [err-question-mark](rules/err-question-mark.md) | Use the question-mark operator for clean propagation |
| [err-result-over-panic](rules/err-result-over-panic.md) | Return Result instead of panicking for recoverable errors |
| [err-source-chain](rules/err-source-chain.md) | Preserve error chains with #[source] |
| [err-thiserror-lib](rules/err-thiserror-lib.md) | Use Thiserror for library error types |

### Ownership & Borrowing [MEDIUM]
Work with the borrow checker instead of against it.

| Rule | Description |
|------|-------------|
| [own-arc-shared](rules/own-arc-shared.md) | Use Arc for thread-safe shared ownership |
| [own-borrow-over-clone](rules/own-borrow-over-clone.md) | Prefer borrowing over cloning |
| [own-clone-explicit](rules/own-clone-explicit.md) | Use explicit Clone for types where copying has a cost |
| [own-copy-small](rules/own-copy-small.md) | Implement Copy for small, simple types |
| [own-cow-conditional](rules/own-cow-conditional.md) | Use Cow for conditional ownership |
| [own-lifetime-elision](rules/own-lifetime-elision.md) | Rely on lifetime elision rules |
| [own-move-large](rules/own-move-large.md) | Move large types via Box instead of copying |
| [own-mutex-interior](rules/own-mutex-interior.md) | Use Mutex for interior mutability across threads |
| [own-rc-single-thread](rules/own-rc-single-thread.md) | Use Rc for shared ownership in single-threaded code |
| [own-refcell-interior](rules/own-refcell-interior.md) | Use RefCell for interior mutability in single-threaded code |
| [own-rwlock-readers](rules/own-rwlock-readers.md) | Use RwLock when reads significantly outnumber writes |
| [own-slice-over-vec](rules/own-slice-over-vec.md) | Accept slices instead of Vec and String references |

### Macros [MEDIUM]
Declarative and procedural macro patterns that stay hygienic and debuggable.

| Rule | Description |
|------|-------------|
| [macro-export-crate-path](rules/macro-export-crate-path.md) | Export declarative macros with macro_export and a clean import path |
| [macro-fragment-specifiers](rules/macro-fragment-specifiers.md) | Capture with precise fragment specifiers, not raw tt |
| [macro-prefer-functions](rules/macro-prefer-functions.md) | Reach for a macro only when a function cannot express it |
| [macro-private-helpers](rules/macro-private-helpers.md) | Hide macro-generated helpers behind a doc hidden __private module |
| [macro-proc-error-spans](rules/macro-proc-error-spans.md) | Report proc-macro errors as spanned compile errors, never by panicking |
| [macro-proc-syn-quote](rules/macro-proc-syn-quote.md) | Build procedural macros with syn, quote, and proc-macro2 |
| [macro-proc-two-crate](rules/macro-proc-two-crate.md) | Put proc-macros in a dedicated crate, re-export from a facade |
| [macro-rules-hygiene](rules/macro-rules-hygiene.md) | Rely on macro_rules hygiene, use $crate for Item paths |

### Traits [MEDIUM]
Trait design choices that balance flexibility, dispatch cost, and coherence.

| Rule | Description |
|------|-------------|
| [trait-associated-type-vs-generic](rules/trait-associated-type-vs-generic.md) | Choose associated types for one output, generics for many |
| [trait-blanket-impl](rules/trait-blanket-impl.md) | Use a blanket impl to cover an entire class of types |
| [trait-coherence-newtype](rules/trait-coherence-newtype.md) | Wrap a foreign type in a newtype to satisfy the orphan rule |
| [trait-default-methods](rules/trait-default-methods.md) | Build traits from a few required methods plus defaults |
| [trait-dyn-vs-generic](rules/trait-dyn-vs-generic.md) | Choose static or dynamic dispatch deliberately |
| [trait-object-safety](rules/trait-object-safety.md) | Keep a trait object-safe when it needs to support dyn |

### Closures [MEDIUM]
Closure capture and trait-bound choices for callback-heavy code.

| Rule | Description |
|------|-------------|
| [closure-disjoint-capture](rules/closure-disjoint-capture.md) | Capture only what you use with disjoint closure captures |
| [closure-fn-trait-bounds](rules/closure-fn-trait-bounds.md) | Require the least restrictive Fn trait a callback needs |
| [closure-impl-fn-return](rules/closure-impl-fn-return.md) | Return closures as impl Fn, not Box dyn Fn |
| [closure-move-capture](rules/closure-move-capture.md) | Use move for closures that outlive the current scope |
| [closure-static-vs-dyn](rules/closure-static-vs-dyn.md) | Choose generic impl Fn or dyn Fn by call site, not by habit |

### Pattern Matching [MEDIUM]
Exhaustive, expressive matching over enums and data structures.

| Rule | Description |
|------|-------------|
| [pat-at-bindings](rules/pat-at-bindings.md) | Use @ bindings to capture while matching |
| [pat-exhaustive-enum](rules/pat-exhaustive-enum.md) | Match owned enums exhaustively, avoid catch-all wildcards |
| [pat-if-let-chains](rules/pat-if-let-chains.md) | Combine bindings and conditions with if let chains |
| [pat-let-else](rules/pat-let-else.md) | Use let else for early-return pattern extraction |
| [pat-matches-macro](rules/pat-matches-macro.md) | Use matches! for boolean pattern tests |

### Collections [MEDIUM]
Choose and use the standard collection types correctly.

| Rule | Description |
|------|-------------|
| [coll-binaryheap](rules/coll-binaryheap.md) | Use BinaryHeap for a priority queue or repeated max-extraction |
| [coll-map-choice](rules/coll-map-choice.md) | Pick the map type by access pattern, not by Default habit |
| [coll-seq-choice](rules/coll-seq-choice.md) | Default to Vec, reach for VecDeque for queue behavior |
| [coll-set-membership](rules/coll-set-membership.md) | Use HashSet or BTreeSet for membership tests, not linear Vec::contains |

### Const & Compile-Time [MEDIUM]
Move work to compile time with const generics, const fn, and static evaluation.

| Rule | Description |
|------|-------------|
| [const-block](rules/const-block.md) | Use const blocks for compile-time evaluation and assertions |
| [const-fn](rules/const-fn.md) | Mark pure functions const fn when they can run at compile time |
| [const-generics](rules/const-generics.md) | Parameterize over sizes with const generics |
| [const-vs-static](rules/const-vs-static.md) | Use const for inlined values, static for a single addressed instance |

### Conversions [MEDIUM]
Type conversion traits that make transformations explicit and fallible where needed.

| Rule | Description |
|------|-------------|
| [conv-asmut-mutable](rules/conv-asmut-mutable.md) | Accept impl AsMut for flexible mutable inputs |
| [conv-fromstr-parsing](rules/conv-fromstr-parsing.md) | Implement FromStr instead of a bespoke parse function |
| [conv-tryfrom-fallible](rules/conv-tryfrom-fallible.md) | Implement TryFrom for fallible conversions |

### Documentation [LOW]
Rustdoc conventions that keep public APIs discoverable and example code compilable.

| Rule | Description |
|------|-------------|
| [doc-all-public](rules/doc-all-public.md) | Write a doc comment for every public Item |
| [doc-cargo-metadata](rules/doc-cargo-metadata.md) | Fill in Cargo.toml metadata before publishing |
| [doc-crate-readme](rules/doc-crate-readme.md) | Derive the crate-root doc comment from the README |
| [doc-errors-section](rules/doc-errors-section.md) | Format the # errors section consistently for fallible functions |
| [doc-examples-section](rules/doc-examples-section.md) | Give every non-trivial API an # examples block |
| [doc-hidden-setup](rules/doc-hidden-setup.md) | Hide doctest setup lines behind a # prefix |
| [doc-intra-links](rules/doc-intra-links.md) | Link to items with intra-doc link syntax, not plain text |
| [doc-link-types](rules/doc-link-types.md) | Cross-Link related types to build a navigable doc graph |
| [doc-module-inner](rules/doc-module-inner.md) | Give every module a //! overview comment |
| [doc-panics-section](rules/doc-panics-section.md) | Document every panic condition in a # panics section |
| [doc-question-mark](rules/doc-question-mark.md) | Propagate errors with ? in doc examples, not unwrap |
| [doc-safety-section](rules/doc-safety-section.md) | Spell out the contract in a # safety section on every unsafe fn |

### Compiler Optimization [LOW]
Codegen and build-level tuning: inlining hints, LTO, PGO, and target features.

| Rule | Description |
|------|-------------|
| [opt-bounds-check](rules/opt-bounds-check.md) | Prefer Iterator patterns that eliminate bounds checks |
| [opt-cache-friendly](rules/opt-cache-friendly.md) | Organize data for cache-efficient access patterns |
| [opt-codegen-units](rules/opt-codegen-units.md) | Set codegen-units = 1 for maximum release optimization |
| [opt-cold-unlikely](rules/opt-cold-unlikely.md) | Mark rarely-taken paths with #[cold] |
| [opt-inline-always-rare](rules/opt-inline-always-rare.md) | Reserve #[inline(always)] for proven hot paths |
| [opt-inline-never-cold](rules/opt-inline-never-cold.md) | Extract error and cold paths with #[inline(never)] |
| [opt-inline-small](rules/opt-inline-small.md) | Use #[inline] for small cross-crate hot functions |
| [opt-likely-hint](rules/opt-likely-hint.md) | Structure branches to hint the likely path |
| [opt-lto-release](rules/opt-lto-release.md) | Enable link-time optimization in release builds |
| [opt-pgo-profile](rules/opt-pgo-profile.md) | Use profile-guided optimization for maximum performance |
| [opt-simd-portable](rules/opt-simd-portable.md) | Reach for portable SIMD before architecture-specific intrinsics |
| [opt-target-cpu](rules/opt-target-cpu.md) | Set target-cpu for known deployment targets |

## Quick Reference

### Async Programming
```rust
// Many concurrent connections: keep the default multi-threaded runtime,
// and size worker_threads deliberately rather than accepting whatever
// the host's core count happens to be.
#[tokio::main(worker_threads = 4)]
async fn main() {
    let client = Client::new();
    client.run().await;
}
```

### Type Safety
```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct UserId(pub u64);

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct PostId(pub u64);

fn get_user_posts(user_id: UserId, post_id: PostId) -> Vec<Post> {
    todo!()
}
```

### Serialization (Serde)
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
#[serde(deny_unknown_fields)]
struct ServerConfig {
    host: String,
    port: u16,
    timeout_secs: u64,
}

// A typo like "timout_secs" is now a hard, actionable deserialize error
// instead of a silently-ignored field.
```

### Observability
```rust
use tracing::info;

fn handle_login(id: u64) {
    info!(user.id = %id, "user logged in"); // filterable, structured, levelled
}

fn main() {
    tracing_subscriber::fmt::init(); // subscriber setup belongs in the binary
    handle_login(42);
}
```

### Unsafe Rust
```rust
/// Returns the byte at `ptr + offset`.
///
/// # Safety
///
/// - `ptr` must be valid for reads for at least `offset + 1` bytes.
pub unsafe fn read_at(ptr: *const u8, offset: usize) -> u8 {
    // SAFETY: caller guarantees ptr is valid for at least offset + 1 bytes,
    // so ptr.add(offset) is in bounds and dereferenceable.
    unsafe { *ptr.add(offset) }
}
```

### Numeric Types
```rust
// checked_add: returns None on overflow -- propagate or handle the error
fn add_score(current: u32, delta: u32) -> Option<u32> {
    current.checked_add(delta)
}

// saturating_add: clamps at the type's bound, no error path needed
fn increment_saturating(c: u8) -> u8 {
    c.saturating_add(1)
}
```

### Concurrency
```rust
use std::thread;

fn parallel_sum(data: &[i64]) -> i64 {
    let mid = data.len() / 2;
    let (left, right) = data.split_at(mid);

    thread::scope(|s| {
        let h1 = s.spawn(|| left.iter().sum::<i64>());
        let h2 = s.spawn(|| right.iter().sum::<i64>());
        h1.join().unwrap() + h2.join().unwrap()
    })
}
```

### API Design
```rust
#[derive(Default)]
#[must_use = "builders do nothing unless you call build()"]
pub struct ClientBuilder {
    base_url: Option<String>,
    timeout: Option<Duration>,
}

impl ClientBuilder {
    pub fn base_url(mut self, url: impl Into<String>) -> Self {
        self.base_url = Some(url.into());
        self
    }

    pub fn build(self) -> Result<Client, BuilderError> {
        let base_url = self.base_url.ok_or(BuilderError::MissingBaseUrl)?;
        Ok(Client { base_url, timeout: self.timeout.unwrap_or(Duration::from_secs(30)) })
    }
}

// Usage -- clear and self-documenting
let client = ClientBuilder::default().base_url("https://api.example.com").build()?;
```

### Memory Management
```rust
// Pre-allocate the exact size: zero reallocations
let mut results = Vec::with_capacity(1000);
for i in 0..1000 {
    results.push(process(i));
}

// Or let collect() use the iterator's size hint automatically
let results: Vec<_> = (0..1000).map(process).collect();
```

### Naming
```rust
fn fetch_user_profile(user_id: u64) -> UserProfile {
    let last_login = user_id;
    todo!()
}

mod user_service;
mod http_client;
```

### Anti-Patterns
```rust
fn summarize(report: &Report) {
    log_report(report);          // read-only, borrow is enough
    if report.is_valid() {       // read-only, no clone needed
        archive(report.clone()); // clone only where ownership is genuinely required
    }
}
```

### Performance
```rust
use std::collections::HashMap;

fn increment(map: &mut HashMap<String, u32>, key: String) {
    *map.entry(key).or_insert(0) += 1; // single lookup either way
}
```

### Error Handling
```rust
fn load_config() -> Result<Config, Error> {
    let content = std::fs::read_to_string("config.toml")?;
    let config = toml::from_str(&content)?;
    Ok(config)
}
```

### Ownership & Borrowing
```rust
fn process(data: &str) { // Accept &str, more flexible
    println!("{}", data); // No allocation needed
}

fn count_words(text: &str) -> usize {
    text.split_whitespace().count() // Just borrow
}
```

### Macros
```rust
#[macro_export]
macro_rules! log {
    ($val:expr) => {
        // $crate always expands to the crate that defined this macro
        $crate::log_value(&format!("{:?}", $val));
    };
}

// Hygiene for local bindings: `tmp` inside the macro never clashes with the caller's `tmp`
macro_rules! swap {
    ($a:expr, $b:expr) => {{
        let tmp = $a;
        $a = $b;
        $b = tmp;
    }};
}
```

### Traits
```rust
use std::fmt;

trait Describe {
    fn describe(&self) -> String;
}

// One impl covers every T: Display, mirroring std's ToString.
impl<T: fmt::Display> Describe for T {
    fn describe(&self) -> String {
        format!("{self} ({})", std::any::type_name::<T>())
    }
}
```

### Closures
```rust
// Call exactly once -- accept FnOnce, the widest possible bound
fn run_once<F: FnOnce() -> String>(f: F) -> String {
    f()
}

// Call multiple times, closure may mutate its captures -- accept FnMut
fn retry<F: FnMut() -> bool>(mut f: F, attempts: usize) -> bool {
    (0..attempts).any(|_| f())
}
```

### Pattern Matching
```rust
fn process(input: Option<String>) -> Option<u32> {
    let Some(s) = input else { return None; };
    let Ok(n) = s.trim().parse::<u32>() else { return None; };
    if n == 0 {
        return None;
    }
    Some(n * 2)
}
```

### Collections
```rust
use std::collections::HashMap;

// HashMap: default, fast, order irrelevant
fn total_scores<'a>(records: &[(&'a str, u32)]) -> HashMap<&'a str, u32> {
    let mut scores: HashMap<&'a str, u32> = HashMap::new();
    for &(name, score) in records {
        *scores.entry(name).or_insert(0) += score;
    }
    scores
}
```

### Const & Compile-Time
```rust
const fn header_len() -> usize {
    4
}

// Usable as an array length -- evaluated at compile time
let buf = [0u8; header_len()];

const fn align_up(n: usize, align: usize) -> usize {
    (n + align - 1) & !(align - 1)
}

const ALIGNED: usize = align_up(13, 8); // 16, computed once, at compile time
```

### Conversions
```rust
#[derive(Debug)]
struct Port(u16);

impl TryFrom<u32> for Port {
    type Error = PortError;

    fn try_from(value: u32) -> Result<Self, Self::Error> {
        u16::try_from(value).map(Port).map_err(|_| PortError(value))
    }
}

fn accept_port(n: u32) -> Result<Port, PortError> {
    n.try_into() // standard idiom, works anywhere TryFrom<u32> is implemented
}
```

### Documentation
```rust
/// Sends the request and parses the response.
///
/// # Errors
///
/// - [`HttpError::Timeout`] if the server doesn't respond within the configured timeout
/// - [`HttpError::InvalidUrl`] if `req`'s URL fails to parse
pub fn send(req: Request) -> Result<Response, HttpError> {
    // ...
}
```

### Compiler Optimization
```rust
#[inline]
pub fn is_ascii_digit(b: u8) -> bool {
    b >= b'0' && b <= b'9'
}
```

## See Also

- [rust-testing](../rust-testing/SKILL.md) - Test-writing best practices for Rust
- [rust-tooling](../rust-tooling/SKILL.md) - clippy, rustfmt, and Cargo workspace/project-structure rules
- [NOTICE](NOTICE.md) - MIT attribution for content adapted from leonardomso/rust-skills
