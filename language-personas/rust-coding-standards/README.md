# Rust Coding Standards

A comprehensive collection of Rust coding standards and best practices, adapted from the leonardomso/rust-skills catalog for AI agents and LLMs to generate idiomatic, safe, and performant Rust code.

## Overview

This skill provides 223 rules across 23 categories:

| Category | Prefix | Impact | Rules |
|----------|--------|--------|-------|
| Async Programming | `async-` | HIGH | 18 |
| Type Safety | `type-` | HIGH | 13 |
| Serialization (Serde) | `serde-` | HIGH | 8 |
| Observability | `obs-` | HIGH | 7 |
| Unsafe Rust | `unsafe-` | HIGH | 7 |
| Numeric Types | `num-` | HIGH | 5 |
| Concurrency | `conc-` | HIGH | 4 |
| API Design | `api-` | MEDIUM | 17 |
| Memory Management | `mem-` | MEDIUM | 17 |
| Naming | `name-` | MEDIUM | 16 |
| Anti-Patterns | `anti-` | MEDIUM | 15 |
| Performance | `perf-` | MEDIUM | 13 |
| Error Handling | `err-` | MEDIUM | 12 |
| Ownership & Borrowing | `own-` | MEDIUM | 12 |
| Macros | `macro-` | MEDIUM | 8 |
| Traits | `trait-` | MEDIUM | 6 |
| Closures | `closure-` | MEDIUM | 5 |
| Pattern Matching | `pat-` | MEDIUM | 5 |
| Collections | `coll-` | MEDIUM | 4 |
| Const & Compile-Time | `const-` | MEDIUM | 4 |
| Conversions | `conv-` | MEDIUM | 3 |
| Documentation | `doc-` | LOW | 12 |
| Compiler Optimization | `opt-` | LOW | 12 |

## Structure

```
skills/rust-coding-standards/
├── SKILL.md              # Skill overview with all rule summaries
├── metadata.json          # Metadata (version, description)
├── NOTICE.md              # MIT attribution to leonardomso/rust-skills
├── README.md              # This file
└── rules/
    ├── _sections.md      # Section definitions
    ├── _template.md      # Rule template
    ├── async-*.md       # Async Programming rules
    ├── type-*.md        # Type Safety rules
    ├── serde-*.md       # Serialization (Serde) rules
    ├── obs-*.md         # Observability rules
    ├── unsafe-*.md      # Unsafe Rust rules
    ├── num-*.md         # Numeric Types rules
    ├── conc-*.md        # Concurrency rules
    ├── api-*.md         # API Design rules
    ├── mem-*.md         # Memory Management rules
    ├── name-*.md        # Naming rules
    ├── anti-*.md        # Anti-Patterns rules
    ├── perf-*.md        # Performance rules
    ├── err-*.md         # Error Handling rules
    ├── own-*.md         # Ownership & Borrowing rules
    ├── macro-*.md       # Macros rules
    ├── trait-*.md       # Traits rules
    ├── closure-*.md     # Closures rules
    ├── pat-*.md         # Pattern Matching rules
    ├── coll-*.md        # Collections rules
    ├── const-*.md       # Const & Compile-Time rules
    ├── conv-*.md        # Conversions rules
    ├── doc-*.md         # Documentation rules
    └── opt-*.md         # Compiler Optimization rules
```

## Rules

### Async Programming (HIGH)
- `async-async-fn-bounds` - Use AsyncFn bounds instead of F: Fn() -> Fut
- `async-bounded-channel` - Use bounded channels to apply backpressure
- `async-broadcast-pubsub` - Use broadcast for Pub/Sub where all subscribers see all messages
- `async-cancel-safety` - Ensure futures in select! branches are cancellation-safe
- `async-cancellation-token` - Use CancellationToken for graceful shutdown
- `async-clone-before-await` - Clone Arc/Rc data before await points
- `async-fn-in-trait` - Prefer native async fn in traits over async-trait
- `async-join-parallel` - Use join! to run independent futures concurrently
- `async-joinset-structured` - Use JoinSet for dynamic collections of spawned tasks
- `async-mpsc-queue` - Use tokio::sync::mpsc for task-to-task message queues
- `async-no-lock-await` - Never hold a Mutex/RwLock guard across an await point
- `async-oneshot-response` - Use oneshot for single request-response replies
- `async-select-racing` - Use select! to race futures and cancel the losers
- `async-spawn-blocking` - Use spawn_blocking for CPU-intensive or blocking work
- `async-tokio-fs` - Use tokio::fs instead of std::fs in async code
- `async-tokio-runtime` - Configure the Tokio runtime for the actual workload
- `async-try-join` - Use try_join! for concurrent fallible operations
- `async-watch-latest` - Use watch for broadcasting the latest value

### Type Safety (HIGH)
- `type-deref-coercion` - Implement Deref only for smart pointers and transparent wrappers
- `type-display-vs-debug` - Use Display for users and Debug for diagnostics — never swap them
- `type-enum-states` - Use enums for mutually exclusive states
- `type-generic-bounds` - Place trait bounds where needed, prefer where clauses
- `type-never-diverge` - Use the never type for functions that never return
- `type-newtype-ids` - Wrap IDs in newtypes instead of raw integers
- `type-newtype-validated` - Use newtypes to enforce validation at construction
- `type-no-stringly` - Avoid stringly-typed APIs
- `type-numeric-fmt` - Implement Hex/Octal/Binary formatting for numeric newtypes
- `type-option-nullable` - Use Option for values that might not exist
- `type-phantom-marker` - Use PhantomData to express type relationships at zero cost
- `type-repr-transparent` - Use #[repr(transparent)] for FFI newtypes
- `type-result-fallible` - Use Result for operations that can fail

### Serialization (Serde) (HIGH)
- `serde-custom-with` - Customize field (De)serialization with serde with modules
- `serde-default-compat` - Use serde default for optional and backward-compatible fields
- `serde-deny-unknown-fields` - Reject unexpected keys with serde deny_unknown_fields
- `serde-enum-representation` - Choose Serde enum tagging deliberately
- `serde-flatten` - Inline shared fields with serde flatten
- `serde-rename-all` - Match external naming with serde rename_all
- `serde-skip-empty` - Omit empty fields with serde skip_serializing_if
- `serde-try-from-validate` - Validate during deserialization with serde try_from

### Observability (HIGH)
- `obs-error-chain` - Log the full error chain once, at the layer that handles it
- `obs-instrument-spans` - Attach async work to spans with #[instrument], never a held guard
- `obs-levels-filter` - Pick log levels by urgency and filter at runtime with EnvFilter
- `obs-library-facade` - Libraries emit through tracing/log, only binaries install a subscriber
- `obs-no-sensitive-data` - Never let secrets or PII reach a log line or span field
- `obs-structured-fields` - Record values as structured fields, not interpolated into the message
- `obs-tracing-over-log` - Reach for tracing instead of println! or bare log for diagnostics

### Unsafe Rust (HIGH)
- `unsafe-extern-block` - Wrap extern blocks in unsafe extern with per-item safety
- `unsafe-maybeuninit` - Use MaybeUninit for uninitialized memory
- `unsafe-minimize-scope` - Keep unsafe blocks as small as possible
- `unsafe-miri-ci` - Run cargo miri test in CI for every unsafe-containing crate
- `unsafe-no-mangle-unsafe` - Write #[unsafe(no_mangle)] instead of bare attribute forms
- `unsafe-safety-comment` - Document every unsafe block with a SAFETY comment
- `unsafe-send-sync-manual` - Document invariants when manually implementing Send or Sync

### Numeric Types (HIGH)
- `num-cast-try-from` - Avoid as for narrowing casts — use TryFrom
- `num-float-compare` - Never compare floats with == — use tolerance or total_cmp
- `num-nonzero` - Use NonZero types to forbid zero at the type level
- `num-overflow-explicit` - Handle integer overflow explicitly
- `num-saturating-clamp` - Bound values with clamp and saturating arithmetic

### Concurrency (HIGH)
- `conc-atomic-ordering` - Use the weakest correct memory ordering for atomics
- `conc-rayon-par-iter` - Use rayon's par_iter for CPU-bound data parallelism
- `conc-scoped-threads` - Use thread::scope to borrow stack data across threads
- `conc-thread-local` - Prefer thread_local! over static mut

### API Design (MEDIUM)
- `api-builder-must-use` - Mark builder methods with #[must_use]
- `api-builder-pattern` - Use the builder pattern for complex construction
- `api-common-traits` - Implement common traits for public types
- `api-default-impl` - Implement Default for types with sensible defaults
- `api-extension-trait` - Use extension traits to add methods to external types
- `api-from-not-into` - Implement from, not into
- `api-impl-asref` - Use AsRef<T> when you only need to borrow
- `api-impl-fromiterator` - Implement FromIterator, Extend, and IntoIterator for collection types
- `api-impl-into` - Accept impl Into<T> for flexible APIs
- `api-must-use` - Mark types and functions #[must_use] when ignoring them is a bug
- `api-newtype-safety` - Use newtypes to prevent mixing semantically different values
- `api-non-exhaustive` - Use #[non_exhaustive] for forward-compatible public types
- `api-operator-overload` - Overload operators only when semantics are obvious
- `api-parse-dont-validate` - Parse into validated types at boundaries
- `api-sealed-trait` - Use sealed traits to prevent external implementations
- `api-serde-optional` - Make Serde a feature flag, not a hard dependency
- `api-typestate` - Use the typestate pattern to encode state machine invariants

### Memory Management (MEDIUM)
- `mem-arena-allocator` - Use arena allocators for batch allocations
- `mem-arrayvec` - Use ArrayVec for fixed-capacity stack collections
- `mem-assert-type-size` - Assert type sizes to guard against silent bloat
- `mem-avoid-format` - Avoid format! when a String literal works
- `mem-box-large-variant` - Box large enum variants to shrink the whole enum
- `mem-boxed-slice` - Use Box<[T]> for fixed-size heap data
- `mem-clone-from` - Use clone_from to reuse allocations when repeatedly cloning
- `mem-compact-string` - Use compact String types for memory-constrained storage
- `mem-drop-order` - Know and control Drop order
- `mem-reuse-collections` - Clear and reuse collections instead of recreating them in loops
- `mem-smaller-integers` - Use appropriately-sized integers to reduce memory footprint
- `mem-smallvec` - Use SmallVec for usually-small collections
- `mem-take-replace` - Use mem::take / mem::replace to move out of &mut without cloning
- `mem-thinvec` - Use ThinVec for nullable collections with minimal overhead
- `mem-with-capacity` - Pre-allocate with with_capacity when size is known
- `mem-write-over-format` - Use write! into existing buffers instead of format! allocations
- `mem-zero-copy` - Use zero-copy patterns with slices and bytes

### Naming (MEDIUM)
- `name-acronym-word` - Capitalize acronyms as ordinary words
- `name-as-free` - Reserve the as_ prefix for free reference conversions
- `name-consts-screaming` - Name constants and statics in SCREAMING_SNAKE_CASE
- `name-crate-no-rs` - Skip the -rs / -rust suffix on crate names
- `name-funcs-snake` - Name functions, methods, and variables in snake_case
- `name-into-ownership` - Use into_ for conversions that consume self
- `name-is-has-bool` - Prefix boolean methods with is_/has_/can_
- `name-iter-convention` - Offer iter/iter_mut/into_iter as a matched set
- `name-iter-method` - Keep Iterator-producing methods to a fixed naming vocabulary
- `name-iter-type-match` - Name Iterator types after the method that produces them
- `name-lifetime-short` - Keep lifetime names short and conventional
- `name-no-get-prefix` - Drop the get_ prefix on plain field accessors
- `name-to-expensive` - Reserve the to_ prefix for conversions that cost something
- `name-type-param-single` - Name generic type parameters with a single uppercase letter
- `name-types-camel` - Name types, traits, and enums in UpperCamelCase
- `name-variants-camel` - Name enum variants in UpperCamelCase

### Anti-Patterns (MEDIUM)
- `anti-clone-excessive` - Recognize cloning where a borrow would have worked
- `anti-collect-intermediate` - Recognize the collect-then-iterate-again anti-pattern
- `anti-empty-catch` - Never let an error vanish into an empty match arm
- `anti-expect-lazy` - Don't reach for expect() just because it has a message
- `anti-format-hot-path` - Notice format! allocating inside a loop
- `anti-index-over-iter` - Notice a for i in 0..len loop where an Iterator was available
- `anti-lock-across-await` - Treat a lock guard held across .await as an immediate red flag
- `anti-over-abstraction` - Don't generalize before a second concrete use case exists
- `anti-panic-expected` - Don't let panic! stand in for an ordinary failure path
- `anti-premature-optimize` - Don't add optimization complexity before a profiler asks for it
- `anti-string-for-str` - Don't narrow a function parameter to &string
- `anti-stringly-typed` - Watch for two &str parameters that can be silently swapped
- `anti-type-erasure` - Reach for Box<dyn Trait> only when types genuinely vary at runtime
- `anti-unwrap-abuse` - Treat .unwrap() in non-test code as a question, not an answer
- `anti-vec-for-slice` - Don't narrow a function parameter to &Vec<T>

### Performance (MEDIUM)
- `perf-ahash` - Swap SipHash for ahash/FxHash when keys aren't attacker-controlled
- `perf-black-box-bench` - Wrap benchmark inputs and outputs in black_box
- `perf-chain-avoid` - Avoid Iterator::chain inside a hot inner loop
- `perf-collect-into` - Reuse a collection's allocation instead of collecting fresh each time
- `perf-collect-once` - Chain Iterator adapters and collect exactly once
- `perf-drain-reuse` - Use drain() to move elements out without losing the allocation
- `perf-entry-api` - Use the entry API instead of check-then-insert on a map
- `perf-extend-batch` - Batch insertions with extend() instead of repeated push()
- `perf-io-buffering` - Wrap frequent Reads/Writes in BufReader/BufWriter
- `perf-iter-lazy` - Keep an Iterator pipeline lazy until the final consumer
- `perf-iter-over-index` - Traverse slices with iterators, not manual index loops
- `perf-profile-first` - Profile before optimizing — don't guess where the time goes
- `perf-release-profile` - Tune [profile.release] instead of shipping cargo's defaults

### Error Handling (MEDIUM)
- `err-anyhow-app` - Use Anyhow for application error handling
- `err-context-chain` - Add context to errors with context() and with_context()
- `err-custom-type` - Define custom error types for domain-specific failures
- `err-doc-errors` - Document error conditions with an errors section
- `err-expect-bugs-only` - Reserve expect() for invariants, not user or external errors
- `err-from-impl` - Implement from for error conversions to enable the question-mark operator
- `err-lowercase-msg` - Start error messages lowercase, no trailing punctuation
- `err-no-unwrap-prod` - Avoid unwrap() in production code
- `err-question-mark` - Use the question-mark operator for clean propagation
- `err-result-over-panic` - Return Result instead of panicking for recoverable errors
- `err-source-chain` - Preserve error chains with #[source]
- `err-thiserror-lib` - Use Thiserror for library error types

### Ownership & Borrowing (MEDIUM)
- `own-arc-shared` - Use Arc for thread-safe shared ownership
- `own-borrow-over-clone` - Prefer borrowing over cloning
- `own-clone-explicit` - Use explicit Clone for types where copying has a cost
- `own-copy-small` - Implement Copy for small, simple types
- `own-cow-conditional` - Use Cow for conditional ownership
- `own-lifetime-elision` - Rely on lifetime elision rules
- `own-move-large` - Move large types via Box instead of copying
- `own-mutex-interior` - Use Mutex for interior mutability across threads
- `own-rc-single-thread` - Use Rc for shared ownership in single-threaded code
- `own-refcell-interior` - Use RefCell for interior mutability in single-threaded code
- `own-rwlock-readers` - Use RwLock when reads significantly outnumber writes
- `own-slice-over-vec` - Accept slices instead of Vec and String references

### Macros (MEDIUM)
- `macro-export-crate-path` - Export declarative macros with macro_export and a clean import path
- `macro-fragment-specifiers` - Capture with precise fragment specifiers, not raw tt
- `macro-prefer-functions` - Reach for a macro only when a function cannot express it
- `macro-private-helpers` - Hide macro-generated helpers behind a doc hidden __private module
- `macro-proc-error-spans` - Report proc-macro errors as spanned compile errors, never by panicking
- `macro-proc-syn-quote` - Build procedural macros with syn, quote, and proc-macro2
- `macro-proc-two-crate` - Put proc-macros in a dedicated crate, re-export from a facade
- `macro-rules-hygiene` - Rely on macro_rules hygiene, use $crate for Item paths

### Traits (MEDIUM)
- `trait-associated-type-vs-generic` - Choose associated types for one output, generics for many
- `trait-blanket-impl` - Use a blanket impl to cover an entire class of types
- `trait-coherence-newtype` - Wrap a foreign type in a newtype to satisfy the orphan rule
- `trait-default-methods` - Build traits from a few required methods plus defaults
- `trait-dyn-vs-generic` - Choose static or dynamic dispatch deliberately
- `trait-object-safety` - Keep a trait object-safe when it needs to support dyn

### Closures (MEDIUM)
- `closure-disjoint-capture` - Capture only what you use with disjoint closure captures
- `closure-fn-trait-bounds` - Require the least restrictive Fn trait a callback needs
- `closure-impl-fn-return` - Return closures as impl Fn, not Box dyn Fn
- `closure-move-capture` - Use move for closures that outlive the current scope
- `closure-static-vs-dyn` - Choose generic impl Fn or dyn Fn by call site, not by habit

### Pattern Matching (MEDIUM)
- `pat-at-bindings` - Use @ bindings to capture while matching
- `pat-exhaustive-enum` - Match owned enums exhaustively, avoid catch-all wildcards
- `pat-if-let-chains` - Combine bindings and conditions with if let chains
- `pat-let-else` - Use let else for early-return pattern extraction
- `pat-matches-macro` - Use matches! for boolean pattern tests

### Collections (MEDIUM)
- `coll-binaryheap` - Use BinaryHeap for a priority queue or repeated max-extraction
- `coll-map-choice` - Pick the map type by access pattern, not by Default habit
- `coll-seq-choice` - Default to Vec, reach for VecDeque for queue behavior
- `coll-set-membership` - Use HashSet or BTreeSet for membership tests, not linear Vec::contains

### Const & Compile-Time (MEDIUM)
- `const-block` - Use const blocks for compile-time evaluation and assertions
- `const-fn` - Mark pure functions const fn when they can run at compile time
- `const-generics` - Parameterize over sizes with const generics
- `const-vs-static` - Use const for inlined values, static for a single addressed instance

### Conversions (MEDIUM)
- `conv-asmut-mutable` - Accept impl AsMut for flexible mutable inputs
- `conv-fromstr-parsing` - Implement FromStr instead of a bespoke parse function
- `conv-tryfrom-fallible` - Implement TryFrom for fallible conversions

### Documentation (LOW)
- `doc-all-public` - Write a doc comment for every public Item
- `doc-cargo-metadata` - Fill in Cargo.toml metadata before publishing
- `doc-crate-readme` - Derive the crate-root doc comment from the README
- `doc-errors-section` - Format the # errors section consistently for fallible functions
- `doc-examples-section` - Give every non-trivial API an # examples block
- `doc-hidden-setup` - Hide doctest setup lines behind a # prefix
- `doc-intra-links` - Link to items with intra-doc link syntax, not plain text
- `doc-link-types` - Cross-Link related types to build a navigable doc graph
- `doc-module-inner` - Give every module a //! overview comment
- `doc-panics-section` - Document every panic condition in a # panics section
- `doc-question-mark` - Propagate errors with ? in doc examples, not unwrap
- `doc-safety-section` - Spell out the contract in a # safety section on every unsafe fn

### Compiler Optimization (LOW)
- `opt-bounds-check` - Prefer Iterator patterns that eliminate bounds checks
- `opt-cache-friendly` - Organize data for cache-efficient access patterns
- `opt-codegen-units` - Set codegen-units = 1 for maximum release optimization
- `opt-cold-unlikely` - Mark rarely-taken paths with #[cold]
- `opt-inline-always-rare` - Reserve #[inline(always)] for proven hot paths
- `opt-inline-never-cold` - Extract error and cold paths with #[inline(never)]
- `opt-inline-small` - Use #[inline] for small cross-crate hot functions
- `opt-likely-hint` - Structure branches to hint the likely path
- `opt-lto-release` - Enable link-time optimization in release builds
- `opt-pgo-profile` - Use profile-guided optimization for maximum performance
- `opt-simd-portable` - Reach for portable SIMD before architecture-specific intrinsics
- `opt-target-cpu` - Set target-cpu for known deployment targets

## Usage

This skill is automatically applied when working with Rust files (`**/*.rs`) and Cargo manifests (`Cargo.toml`).

## Rule Format

Each rule follows a consistent format:
- **Frontmatter**: title, impact level, tags
- **Description**: Why this rule matters
- **Bad Example**: Code to avoid
- **Good Example**: Recommended approach
- **Notes**: Additional tips and edge cases
- **References**: Documentation links

## Contributing

To add a new rule:
1. Copy `rules/_template.md`
2. Use the appropriate prefix (`async-`, `type-`, `serde-`, `obs-`, `unsafe-`, `num-`, `conc-`, `api-`, `mem-`, `name-`, `anti-`, `perf-`, `err-`, `own-`, `macro-`, `trait-`, `closure-`, `pat-`, `coll-`, `const-`, `conv-`, `doc-`, `opt-`)
3. Follow the existing format
4. Update `_sections.md` and `SKILL.md`
5. If the rule is adapted from leonardomso/rust-skills, keep the attribution in `NOTICE.md` accurate; original rules don't need a NOTICE.md update
