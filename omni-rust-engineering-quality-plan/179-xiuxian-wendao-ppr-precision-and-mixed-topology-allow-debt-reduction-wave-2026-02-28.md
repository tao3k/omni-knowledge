# 179. Xiuxian-Wendao PPR Precision and Mixed Topology Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_ppr_weight_precision.rs`
  - `tests/test_mixed_graph_topology.rs`

## Why This Wave

These two graph-behavior tests are compact and well-scoped, suitable for
continued suppression-debt reduction while keeping risk low.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_ppr_weight_precision.rs`
   - `tests/test_mixed_graph_topology.rs`

2. Root-cause cleanup:
   - `test_ppr_weight_precision.rs`
     - `format!("{}.md", id)` -> `format!("{id}.md")`
     - inlined assertion format args (`{pb}`, `{pd}`, `{stems:?}`)
     - `doc_markdown` fix: wrapped `HippoRAG` in backticks
   - `test_mixed_graph_topology.rs`
     - removed unnecessary raw-string hashes for fixture markdown
     - removed unnecessary allocation in contains checks by switching to
       iterator predicate checks

No new lint suppressions were introduced.

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

- Result: pass
- Note: one intermediate `doc_markdown` warning on `HippoRAG` was fixed and
  rerun to clean pass.

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `22`
  - After this wave: `20`
  - Net reduction: `2` files

## Engineering Outcome

- Weighted-seed PPR precision and mixed topology tests are now suppression-free.
- Remaining debt is concentrated in seed-priors and CLI agentic scenario suites.
