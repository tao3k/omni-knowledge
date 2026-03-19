# 176. Xiuxian-Wendao Intent and LinkGraph-Refs Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_intent.rs`
  - `tests/test_link_graph_refs.rs`

## Why This Wave

These two focused unit-test files were small, low-coupling candidates for
continued suppression-debt burndown in the `xiuxian-wendao` test lane.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_intent.rs`
   - `tests/test_link_graph_refs.rs`

2. Root-cause fix:
   - `tests/test_link_graph_refs.rs`
     - fixed `doc_markdown` by changing module docs from `LinkGraph` to
       `` `LinkGraph` ``.

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
- Note: one intermediate `doc_markdown` warning was fixed and rerun to clean
  pass.

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `26`
  - After this wave: `24`
  - Net reduction: `2` files

## Engineering Outcome

- Query-intent and LinkGraph-ref tests are now suppression-free under strict
  pedantic checks.
- Remaining debt is concentrated in larger benchmark, topology, and CLI
  scenario suites.
