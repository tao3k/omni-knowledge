# Xiuxian-Wendao Sync Tests Suppression Cleanup (2026-02-26)

## Scope

This shard records targeted cleanup for `xiuxian-wendao` sync unit tests,
removing suppression-based lint handling in favor of root-cause test code
improvements.

Target:

- `packages/rust/crates/xiuxian-wendao/src/sync/tests.rs`

## Changes Implemented

### 1) Removed file-level suppression

Actions:

- Deleted:
  - `#![allow(clippy::expect_used, clippy::map_unwrap_or)]`

### 2) Converted panic-style test flow to fallible test flow

Actions:

- Added shared test alias:
  - `type TestResult = std::result::Result<(), Box<dyn std::error::Error>>;`
- Updated file-system touching tests to return `TestResult`.
- Replaced all `expect(...)` calls with `?`.

### 3) Replaced `map(...).unwrap_or(false)` patterns

Actions:

- Updated option predicates to idiomatic `is_some_and(...)`.
- Kept assertions and behavioral intent unchanged.

## Verification Evidence

Executed and passed:

```bash
cargo clippy -p xiuxian-wendao --lib -- -W clippy::pedantic
cargo test -p xiuxian-wendao --lib sync::tests::
```

## Outcome

- Sync test module no longer relies on suppression for `expect_used` and
  `map_unwrap_or`.
- Targeted sync tests remain green (`5/5`).
