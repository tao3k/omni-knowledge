# Xiuxian-Wendao Dependency Indexer Test Suppression Cleanup (2026-02-26)

## Scope

This shard records a focused suppression-debt cleanup wave for
`xiuxian-wendao` dependency-indexer test modules.

Targets:

- `packages/rust/crates/xiuxian-wendao/src/dependency_indexer/cargo/tests.rs`
- `packages/rust/crates/xiuxian-wendao/src/dependency_indexer/symbols/tests.rs`
- `packages/rust/crates/xiuxian-wendao/src/dependency_indexer/indexer/tests.rs`

## Changes Implemented

### 1) Removed file-level suppression attributes

Actions:

- Deleted all file-level `allow(clippy::...)` attributes in these three test
  modules (`expect_used`, `map_unwrap_or`, `uninlined_format_args`).

### 2) Migrated panic-style tests to fallible tests

Actions:

- Added `TestResult` aliases and converted file-system-heavy tests to
  `Result<(), Box<dyn Error>>`.
- Replaced `expect(...)` with `?` across temp-file creation, writes, and parse
  operations.
- Preserved test behavior and assertions.

### 3) Pedantic style normalization

Actions:

- Replaced `map(...).unwrap_or(false)` with `is_some_and(...)`.
- Updated format strings to inline arguments where pedantic recommends it.
- Removed panic-style fallback for workspace-root discovery in
  performance test; now uses explicit error propagation.

## Verification Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao --lib -- -W clippy::pedantic
cargo test -p xiuxian-wendao --lib dependency_indexer::
```

Result:

- Dependency-indexer test lane passed (`12/12`).

## Outcome

- Dependency-indexer test modules no longer rely on suppression attributes.
- Strict pedantic quality remains green for the `xiuxian-wendao` library lane.
