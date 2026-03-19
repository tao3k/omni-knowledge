# 469. Xiuxian Memory Engine Common Test Support Modularization

Date: 2026-03-08

## Scope

This shard records the cleanup of the shared integration-test helper module in
`packages/rust/crates/xiuxian-memory-engine/tests/common/`.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-memory-engine/tests/common/mod.rs` contained the
actual helper implementation for test-store path generation.

That violated the repository rule that `mod.rs` should remain interface-only and
forced the common helper namespace to mix module-root and implementation
responsibility.

## What Changed

Moved the path helper implementation into a focused child module:

- `packages/rust/crates/xiuxian-memory-engine/tests/common/paths.rs`

Reduced `packages/rust/crates/xiuxian-memory-engine/tests/common/mod.rs` to a
pure interface layer:

- `mod paths;`
- `pub(crate) use paths::test_store_path;`

Existing call sites such as `common::test_store_path(...)` remain unchanged.

## Architectural Takeaways

- Test support modules should follow the same modular boundaries as production
  Rust modules.
- Even a single helper belongs in a named child module when the parent is a
  module root.
- Re-exporting stable helper APIs from `mod.rs` preserves ergonomics without
  sacrificing module clarity.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-memory-engine --tests
cargo nextest run -p xiuxian-memory-engine --no-fail-fast
cargo clippy -p xiuxian-memory-engine -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-memory-engine --tests` passed.
- `cargo nextest run -p xiuxian-memory-engine --no-fail-fast` passed
  (`69 passed, 0 skipped`).
- `cargo clippy -p xiuxian-memory-engine -- -W clippy::too_many_lines` passed
  with no new warnings from this cleanup.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-memory-engine/tests/common/mod.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/common/paths.rs`
