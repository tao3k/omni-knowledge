# 514. Xiuxian Zhenfa Tool Macro Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of `test_zhenfa_tool_macro.rs` in
`xiuxian-zhenfa`.

## Why This Change Was Needed

The original file combined generated tool definitions, shared cache fixtures,
macro dispatch assertions, invalid-argument behavior, and cache hit behavior in
one entrypoint. That structure made the macro suite harder to read and extend.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-zhenfa/tests/test_zhenfa_tool_macro.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-zhenfa/tests/test_zhenfa_tool_macro/` with
focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for generated tools and shared cache helpers,
- `dispatch.rs` for successful dispatch and schema assertions,
- `invalid_args.rs` for typed-argument validation behavior,
- `cache.rs` for orchestrator cache-key behavior.

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

- Macro-generated fixtures should live in a dedicated support module so behavior
  tests can stay small and focused.
- Cache behavior and invalid-argument behavior should each have their own test
  module rather than expanding the launcher file.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-zhenfa/tests/test_zhenfa_tool_macro.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_zhenfa_tool_macro/mod.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_zhenfa_tool_macro/support.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_zhenfa_tool_macro/dispatch.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_zhenfa_tool_macro/invalid_args.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_zhenfa_tool_macro/cache.rs`
