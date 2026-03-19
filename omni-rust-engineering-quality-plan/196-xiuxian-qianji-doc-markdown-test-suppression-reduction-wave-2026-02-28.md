# 196. Xiuxian-Qianji Doc-Markdown Test Suppression Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Focus:
  - `tests/test_agenda_validation_pipeline.rs`
  - `tests/test_memory_promotion_pipeline.rs`
  - `tests/test_wendao_ingester_mechanism.rs`
  - `tests/test_qianji_qianhuan_binding.rs`

## Why This Wave

After wave `195`, four remaining test files used
`#![allow(missing_docs, clippy::doc_markdown)]` without needing structural
suppression. This wave removes those allowances and replaces them with
documentation-first entry headers.

## Changes Implemented

1. Removed file-level suppressions from all four files.

2. Added explicit module documentation at file top:
   - agenda validation pipeline integration tests
   - memory promotion pipeline integration tests
   - Wendao ingester integration tests
   - Qianhuan binding contract/runtime tests

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
  - Before this wave: `12` files
  - After this wave: `8` files
  - Net reduction: `4` files

## Engineering Outcome

- `xiuxian-qianji` test suppression debt is now concentrated in eight mixed
  files that also include `unused_imports`.
- The next wave can focus on root-cause cleanup of those mixed files rather
  than doc-only suppressions.
