# 489. Xiuxian Wendao HMAS Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `test_hmas.rs`
integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original HMAS validation test file mixed valid and invalid blackboard
contracts in one top-level implementation file:

- one success-path validation contract,
- multiple distinct failure contracts for missing digital thread, invalid JSON,
  and invalid digital-thread fields.

Those cases are related, but success and failure behavior should not remain
combined in one test module.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_hmas.rs` so it now acts
as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_hmas/` with focused
modules:

- `mod.rs` for the module graph only,
- `support.rs` for the canonical valid blackboard payload fixture,
- `success.rs` for the valid-report contract,
- `failure.rs` for missing-thread, invalid JSON, and invalid-field contracts.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_hmas --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_hmas --no-fail-fast`
  passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Validation suites benefit from splitting success-path and failure-path
  contracts into separate modules even when the total test count is small.
- Canonical valid payloads should live in a shared support module so every test
  uses the same authoritative fixture.
- Thin entrypoints keep the HMAS suite aligned with the broader package-level
  test structure.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_hmas.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_hmas/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_hmas/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_hmas/success.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_hmas/failure.rs`
