# 195. Xiuxian-Qianji Missing-Docs-Only Test Suppression Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Focus:
  - `tests/test_formal_adversarial_audit.rs`
  - `tests/test_smart_commit_integration.rs`
  - `tests/test_schema_contracts.rs`
  - `tests/llm_augmented_formal_audit.rs`
  - `tests/test_scheduler_checkpoint.rs`

## Why This Wave

These five test files used single-purpose `#![allow(missing_docs)]`
suppressions. They are low-risk candidates for suppression debt reduction
through explicit file-level documentation.

## Changes Implemented

1. Removed `#![allow(missing_docs)]` from all five files.

2. Added explicit module docs for four entry files:
   - formal adversarial audit tests
   - smart-commit integration tests
   - schema contract tests
   - scheduler checkpoint tests

3. Kept the existing module docs in
   `llm_augmented_formal_audit.rs` and removed the suppression line.

No new suppressions were introduced.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-qianji
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-qianji cargo clippy -p xiuxian-qianji --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-qianji cargo nextest run -p xiuxian-qianji
```

- Result: pass
- Summary: `45 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-qianji/tests -g '*.rs'`
  - Before this wave: `17` files
  - After this wave: `12` files
  - Net reduction: `5` files

## Engineering Outcome

- `xiuxian-qianji` suppression debt is reduced without touching mixed
  `unused_imports`/`doc_markdown` files yet.
- This creates a cleaner baseline for the next wave focused on mixed-lint
  root-cause fixes.
