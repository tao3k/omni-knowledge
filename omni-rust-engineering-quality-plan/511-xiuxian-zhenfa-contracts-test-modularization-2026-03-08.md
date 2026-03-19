# 511. Xiuxian Zhenfa Contracts Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of `test_contracts.rs` in
`xiuxian-zhenfa`.

## Why This Change Was Needed

The original integration test mixed request validation and response envelope
assertions in one entrypoint. Those contracts evolve independently and should be
validated in separate modules.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-zhenfa/tests/test_contracts.rs` so it now
acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-zhenfa/tests/test_contracts/` with focused
modules:

- `mod.rs` for the module graph only,
- `request.rs` for JSON-RPC request validation rules,
- `response.rs` for JSON-RPC response envelope behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-zhenfa --tests
cargo nextest run -p xiuxian-zhenfa --no-fail-fast
cargo clippy -p xiuxian-zhenfa -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-zhenfa --tests` passed.
- `cargo nextest run -p xiuxian-zhenfa --no-fail-fast` passed (`32 passed, 0 skipped`).
- `cargo clippy -p xiuxian-zhenfa -- -W clippy::too_many_lines` passed.

Notes:

- `cargo check` emitted unrelated `missing-docs` warnings for
  `packages/rust/crates/xiuxian-zhenfa/tests/test_client.rs` and
  `packages/rust/crates/xiuxian-zhenfa/tests/test_gateway.rs`; both files were
  intentionally left untouched because they already contain in-flight user
  edits.

## Architectural Takeaways

- Request and response envelopes deserve separate test modules even when both
  are part of the same RPC contract surface.
- Thin contract launchers make future protocol additions less likely to bloat a
  single file.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-zhenfa/tests/test_contracts.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_contracts/mod.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_contracts/request.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_contracts/response.rs`
