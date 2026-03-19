# 197. Xiuxian-Qianji Test Allow-Debt Zero Convergence Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Focus:
  - `tests/test_qianji_trinity_integration.rs`
  - `tests/test_qianji_precision_research.rs`
  - `tests/unit_qianji_execution.rs`
  - `tests/test_qianji_yaml_orchestration.rs`
  - `tests/unit_qianji_safety.rs`
  - `tests/test_probabilistic_routing.rs`
  - `tests/unit_adversarial_loop.rs`
  - `tests/test_qianji_master_research.rs`

## Why This Wave

After wave `196`, the remaining suppression debt in `xiuxian-qianji/tests`
was concentrated in eight mixed files
(`missing_docs + unused_imports + doc_markdown`). This wave removes those
file-level suppressions and resolves resulting lint feedback directly.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from all eight files.

2. Added explicit module docs to each test entry file to satisfy
   documentation requirements.

3. Fixed the only surfaced root-cause warning:
   - removed unused import `QianjiEngine` from
     `tests/test_qianji_precision_research.rs`.

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

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-qianji/tests -g '*.rs' | wc -l`
  - Before this wave: `8`
  - After this wave: `0`
  - Net reduction: `8` files

## Engineering Outcome

- `xiuxian-qianji/tests` now has zero file-level `#![allow(...)]` debt.
- Remaining quality work can focus on behavioral assertions and architecture,
  not suppression maintenance.
