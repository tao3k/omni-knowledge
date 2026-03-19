# 509. Xiuxian Zhenfa Error Mapping Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the compact but mixed
`test_error_mapping.rs` integration test in `xiuxian-zhenfa`.

## Why This Change Was Needed

Even though the file was small, it still mixed two distinct error domains in one
entrypoint:

- `ZhenfaError` LLM-safe summaries,
- `ZhenfaTransmuterError` LLM-safe summaries.

The workspace standard is to keep integration-test entrypoints thin and move
feature-specific assertions into focused directory modules.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-zhenfa/tests/test_error_mapping.rs` so it
now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-zhenfa/tests/test_error_mapping/` with
focused modules:

- `mod.rs` for the module graph only,
- `zhenfa.rs` for `ZhenfaError` message coverage,
- `transmuter.rs` for `ZhenfaTransmuterError` message coverage.

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

- Small test files still benefit from concern-based module boundaries when they
  mix separate domains.
- Error-model coverage should mirror the domain split of the production error
  types rather than staying bundled in one generic entrypoint.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-zhenfa/tests/test_error_mapping.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_error_mapping/mod.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_error_mapping/zhenfa.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_error_mapping/transmuter.rs`
