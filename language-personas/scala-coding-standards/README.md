# Scala Coding Standards

A comprehensive collection of Scala coding standards and best practices, designed for AI agents and LLMs to generate high-quality, idiomatic, and maintainable Scala code.

## Overview

This skill provides 62 rules across 13 categories:

| Category | Prefix | Impact | Rules |
|----------|--------|--------|-------|
| Null & Option Handling | `null-option-` | CRITICAL | 5 |
| Effect Systems | `effect-systems-` | HIGH | 6 |
| Anti-Patterns | `anti-patterns-` | HIGH | 5 |
| Concurrency | `concurrency-` | HIGH | 5 |
| Error Handling | `error-handling-` | HIGH | 5 |
| Immutability | `immutability-` | HIGH | 5 |
| Implicits & Givens | `implicits-givens-` | MEDIUM | 6 |
| API Design | `api-design-` | MEDIUM | 5 |
| Collections | `collections-` | MEDIUM | 5 |
| Pattern Matching | `pattern-matching-` | MEDIUM | 5 |
| For-Comprehensions | `for-comprehensions-` | MEDIUM | 4 |
| Naming | `naming-` | LOW | 3 |
| Scaladoc | `scaladoc-` | LOW | 3 |

## Structure

```
skills/scala-coding-standards/
├── SKILL.md              # Skill overview with all rule summaries
├── metadata.json         # Metadata (version, description)
├── README.md             # This file
└── rules/
    ├── _sections.md      # Section definitions
    ├── _template.md      # Rule template
    ├── null-option-*.md        # Null & Option Handling rules
    ├── effect-systems-*.md     # Effect Systems rules
    ├── anti-patterns-*.md      # Anti-Patterns rules
    ├── concurrency-*.md        # Concurrency rules
    ├── error-handling-*.md     # Error Handling rules
    ├── immutability-*.md       # Immutability rules
    ├── implicits-givens-*.md   # Implicits & Givens rules
    ├── api-design-*.md         # API Design rules
    ├── collections-*.md        # Collections rules
    ├── pattern-matching-*.md   # Pattern Matching rules
    ├── for-comprehensions-*.md # For-Comprehensions rules
    ├── naming-*.md              # Naming rules
    └── scaladoc-*.md            # Scaladoc rules
```

## Rules

### Null & Option Handling (CRITICAL)
- `null-option-avoid-option-get` - Never call .get on an Option
- `null-option-either-for-expected-failure` - Use Either for operations that can fail with a reason
- `null-option-no-null-literal` - Never use null as a value
- `null-option-option-for-absence` - Represent optional values with Option[T]
- `null-option-orelse-chains-over-nested-match` - Chain orElse instead of nesting match on Option

### Effect Systems (HIGH)
- `effect-systems-direct-style-ox` - Use Ox's direct-style supervised/fork for structured concurrency
- `effect-systems-effect-boundary-at-edges` - Keep IO at the edges; keep core logic pure
- `effect-systems-io-for-side-effects` - Wrap side effects in IO instead of running them directly
- `effect-systems-no-blocking-inside-effect` - Never block a thread inside a non-blocking effect
- `effect-systems-referential-transparency` - Keep effect-returning functions referentially transparent
- `effect-systems-resource-safety` - Acquire and release resources with Resource, not manual try/finally

### Anti-Patterns (HIGH)
- `anti-patterns-no-asinstanceof-casts` - Never use asInstanceOf/isInstanceOf; pattern match instead
- `anti-patterns-no-implicit-overuse` - Don't pile every dependency into using parameters
- `anti-patterns-no-null-return` - A collection-returning method returns empty, never null
- `anti-patterns-no-throw-in-pure-code` - A total-looking function signature must not throw
- `anti-patterns-no-var-escape` - Never let a local var escape the scope that declared it

### Concurrency (HIGH)
- `concurrency-avoid-shared-mutable-state` - Never share a var or mutable collection across concurrent tasks
- `concurrency-cancellation-safety` - Protect multi-step mutations with uncancelable, not ad hoc hope
- `concurrency-execution-context-explicit` - Require an ExecutionContext explicitly, don't import a default one
- `concurrency-future-vs-io` - Prefer IO over Future for new concurrent code
- `concurrency-structured-concurrency-ox` - Bound every concurrent fork to a structured scope

### Error Handling (HIGH)
- `error-handling-either-for-domain-errors` - Model domain errors with Either[E, A]
- `error-handling-io-for-effectful-errors` - Represent effectful, failable work with IO
- `error-handling-no-throw-for-control-flow` - Never use throw for ordinary control flow
- `error-handling-try-for-exception-boundaries` - Wrap exception-throwing calls in Try at the boundary
- `error-handling-typed-errors-over-strings` - Model errors as typed ADTs, not strings

### Immutability (HIGH)
- `immutability-avoid-mutable-builder-escape` - Never let a mutable builder escape its construction scope
- `immutability-case-class-for-data` - Model data with case class, not plain classes
- `immutability-copy-method-for-updates` - Use copy to produce updated instances
- `immutability-immutable-collections-default` - Default to immutable collections
- `immutability-val-over-var` - Prefer val over var

### Implicits & Givens (MEDIUM)
- `implicits-givens-avoid-implicit-conversions` - Avoid implicit conversions; make the conversion explicit
- `implicits-givens-context-bounds-syntax` - Use context bound syntax for a single using type class parameter
- `implicits-givens-explicit-imports-no-wildcard` - Import givens by type, not with a blanket wildcard
- `implicits-givens-extension-methods-over-implicit-class` - Use extension methods, not implicit class, to add methods to an existing type
- `implicits-givens-scala3-syntax` - Use given/using, not Scala 2 implicit, for new code
- `implicits-givens-type-class-pattern` - Model capabilities as type classes with given instances

### API Design (MEDIUM)
- `api-design-companion-object-factories` - Construct through companion object factories, not a public constructor
- `api-design-minimal-public-surface` - Keep the public surface to what callers actually need
- `api-design-opaque-types-for-domain-primitives` - Wrap domain primitives in opaque types, not raw String/Long/Int
- `api-design-smart-constructors` - Enforce invariants with a smart constructor, not a public constructor
- `api-design-trait-for-abstraction-not-reuse` - Use a trait to declare an abstraction, not to share an implementation

### Collections (MEDIUM)
- `collections-avoid-mutable-builder-escape` - Never return a collection builder before calling .result()
- `collections-immutable-by-default` - Default to structure-sharing immutable collections
- `collections-lazylist-for-infinite-sequences` - Use LazyList for self-referential or infinite sequences
- `collections-prefer-map-filter-over-loops` - Prefer map/filter/fold over hand-written loops
- `collections-view-for-lazy-chains` - Use .view to avoid materializing intermediate collections in a chain

### Pattern Matching (MEDIUM)
- `pattern-matching-destructuring-over-accessors` - Destructure with pattern matching instead of chained accessors
- `pattern-matching-guard-clauses` - Use guard clauses inside match instead of nested conditionals
- `pattern-matching-match-over-if-else-chains` - Prefer match over long if/else if chains
- `pattern-matching-no-catch-all-unless-intentional` - Don't add a case _ catch-all unless the fallback is genuinely intentional
- `pattern-matching-sealed-trait-exhaustiveness` - Model closed hierarchies with sealed trait/enum for exhaustive match

### For-Comprehensions (MEDIUM)
- `for-comprehensions-avoid-nested-flatmap` - Replace nested flatMap chains with a for-comprehension
- `for-comprehensions-early-exit-with-either` - Use for-comprehensions over Either to short-circuit a validation pipeline
- `for-comprehensions-monadic-composition` - Use for-comprehensions to compose monadic operations
- `for-comprehensions-readability-over-cleverness` - Don't force a for-comprehension where a direct call reads better

### Naming (LOW)
- `naming-lowercase-for-values` - Use lowerCamelCase for values, methods, and parameters
- `naming-symbolic-methods-sparingly` - Reserve symbolic method names for well-established operators
- `naming-uppercase-for-types` - Use UpperCamelCase for classes, traits, objects, and type parameters

### Scaladoc (LOW)
- `scaladoc-example-tags-for-complex-apis` - Show a complex generic API's usage with @example, not prose alone
- `scaladoc-no-signature-narration` - Never restate a parameter's type or name in its own description
- `scaladoc-public-api-contract` - Document the contract, not a restatement of the implementation

## Usage

This skill is automatically applied when working with Scala files (`**/*.scala`) and build files (`build.sbt`, `build.mill`).

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
2. Use the appropriate prefix (`null-option-`, `effect-systems-`, `anti-patterns-`, `concurrency-`, `error-handling-`, `immutability-`, `implicits-givens-`, `api-design-`, `collections-`, `pattern-matching-`, `for-comprehensions-`, `naming-`, `scaladoc-`)
3. Follow the existing format
4. Update `_sections.md` and `SKILL.md`
