# 166. Xiuxian-Wendao Sync Test Lane Allow-Debt Reduction and Pedantic Fixes Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_sync/*.rs`
  - `tests/test_sync/mod.rs`

## Why This Wave

After the `test_knowledge` lane converged, the next compact lane was
`test_sync`, where eight leaf files still had file-level suppression blocks.

## Changes Implemented

Removed file-level `#![allow(...)]` from all sync leaf tests:

- `batch_diff_computation.rs`
- `compute_diff.rs`
- `compute_hash.rs`
- `custom_discovery_options.rs`
- `deleted_files_detection.rs`
- `discover_files.rs`
- `manifest_load_save.rs`
- `skip_hidden_and_directories.rs`

Root-cause clippy fixes after suppression removal:

- `tests/test_sync/batch_diff_computation.rs`
  - `format!("file_{}.py", i)` -> `format!("file_{i}.py")`
  - `format!("content {}", i)` -> `format!("content {i}")`
- `tests/test_sync/mod.rs`
  - Doc comment: `SyncEngine` wrapped in backticks for `doc_markdown`.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-wendao
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass (exit code `0`)

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- First run: transient benchmark failure in
  `test_symbols_benchmark::mixed_symbol_extraction_performance`
  (`12.31s > 3s` threshold).
- Second run (same command): pass.
- Final summary: `286 passed`, `0 failed`, `1 skipped`
- Final run time: `~7.765s`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `54`
  - After this wave: `46`
  - Net reduction: `8` files

## Engineering Outcome

- Entire `test_sync` lane is now suppression-free.
- The benchmark path shows residual runtime variance; existing hard thresholds
  remain a potential source of occasional non-functional flakes.

## Next Slice

- Continue with `tests/test_graph/*` lane suppression-debt reduction.
