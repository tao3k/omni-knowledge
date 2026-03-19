# 457. Xiuxian Wendao Sync Suite Deduplication And Mod Interface Restoration

Date: 2026-03-07

## Scope

This shard records the consolidation of the `xiuxian-wendao` sync test suite
onto one split integration binary and the restoration of `mod.rs` to
interface-only responsibility.

## Why This Change Was Needed

The repository carried two parallel sync test surfaces:

- `packages/rust/crates/xiuxian-wendao/tests/sync_unit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/`

The first duplicated several behaviors already covered by the second:

- manifest load/save,
- hash stability,
- file discovery,
- diff computation,
- hidden-file skipping.

At the same time, `sync_unit.rs` still held four behaviors that had not yet been
migrated into the split suite:

- incremental policy extension support,
- glob pattern extension extraction,
- brace-glob extension extraction,
- compound suffix and explicit-extension precedence.

The split suite also used `super::*` imports sourced from `test_sync/mod.rs`,
which meant the module root was doing more than interface declaration.

## What Changed

### 1) Consolidated all sync behaviors into the split suite

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_sync/incremental_policy.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/extract_extensions.rs`

These files absorb the four unique behaviors that previously lived only in
`sync_unit.rs`.

### 2) Removed the duplicate monolithic sync test file

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/sync_unit.rs`

The sync behavior family now has one authoritative test binary:

- `packages/rust/crates/xiuxian-wendao/tests/test_sync.rs`

### 3) Restored `test_sync/mod.rs` to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_sync/mod.rs`

The module root now only declares child modules.

### 4) Localized dependencies inside each sync test file

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_sync/batch_diff_computation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/compute_diff.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/custom_discovery_options.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/deleted_files_detection.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/discover_files.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/manifest_load_save.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/skip_hidden_and_directories.rs`

Each test file now imports its own dependencies instead of relying on `super::*`
plumbing from `mod.rs`.

## Architectural Takeaways

- A split suite and a monolithic suite should not coexist for the same feature
  family. Pick one authoritative surface and migrate the remaining unique
  behaviors into it.
- `mod.rs` should declare modules, not act as an ambient import bucket.
- Local imports improve scanability and keep test dependencies honest.
- Structural cleanup can reduce duplication without reducing coverage when the
  migration is done behavior-first.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_sync/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/batch_diff_computation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/compute_diff.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/custom_discovery_options.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/deleted_files_detection.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/discover_files.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/manifest_load_save.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/skip_hidden_and_directories.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/incremental_policy.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_sync/extract_extensions.rs`
- `packages/rust/crates/xiuxian-wendao/tests/sync_unit.rs` (removed)

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_sync --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ... --tests` completed cleanly.
- The full `test_sync` binary passed (`13 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Authoritative sync test root:
  - `packages/rust/crates/xiuxian-wendao/tests/test_sync/`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/457-xiuxian-wendao-sync-suite-deduplication-and-mod-interface-restoration-2026-03-07.md`
