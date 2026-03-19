# Xiuxian-Wendao Storage Tests Expect Cleanup (2026-02-26)

## Scope

This shard records a focused root-cause cleanup in `xiuxian-wendao` storage
unit tests to remove panic-style test patterns and align with strict pedantic
quality expectations.

Target:

- `packages/rust/crates/xiuxian-wendao/src/storage/tests.rs`

## Changes Implemented

### 1) Removed `expect_used` suppression

Actions:

- Deleted file-level suppression:
  - `#![allow(clippy::expect_used)]`

### 2) Converted tests from panic-style to fallible style

Actions:

- Added shared alias:
  - `type TestResult = std::result::Result<(), Box<dyn std::error::Error>>;`
- Updated all `#[tokio::test]` functions to return `TestResult`.
- Replaced all `expect(...)` calls with `?` propagation.
- Preserved existing skip behavior for Valkey-dependent tests:
  - tests return `Ok(())` when Valkey URL is not configured.

### 3) Kept behavior unchanged while improving quality signals

Actions:

- Assertions and test intent remain unchanged.
- Only error-flow mechanics and lint-compliance style were adjusted.

## Verification Evidence

Executed and passed:

```bash
cargo clippy -p xiuxian-wendao --lib -- -W clippy::pedantic
cargo test -p xiuxian-wendao --lib storage::tests::
```

## Outcome

- `xiuxian-wendao` storage unit tests no longer rely on `expect` suppression.
- Pedantic clippy remains clean for the library target in this workspace run.
- All targeted storage tests passed (`4/4`).
