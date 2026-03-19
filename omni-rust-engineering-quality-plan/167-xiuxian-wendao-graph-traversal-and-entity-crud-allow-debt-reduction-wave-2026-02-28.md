# 167. Xiuxian-Wendao Graph Traversal and Entity CRUD Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_graph/entity_relation_crud.rs`
  - `tests/test_graph/graph_traversal.rs`

## Why This Wave

After sync-lane convergence, `test_graph` had multiple medium/large files with
file-level suppression markers. This wave starts that lane with two focused
files to keep risk controlled while preserving momentum.

## Changes Implemented

Removed file-level `#![allow(...)]` from:

- `entity_relation_crud.rs`
- `graph_traversal.rs`

Root-cause fixes surfaced by strict clippy:

- `tests/test_graph/graph_traversal.rs`
  - `format!("concept:{}", name)` -> `format!("concept:{name}")`
  - `format!("Concept {}", name)` -> `format!("Concept {name}")`
  - Assertion message formatting switched to inline style:
    `"… Got: {names:?}"`.

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

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`
- Run time: `~62.776s`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `46`
  - After this wave: `44`
  - Net reduction: `2` files

## Engineering Outcome

- `test_graph` lane suppression burndown has started with a clean,
  suppression-free subset.
- Remaining debt is concentrated in other graph files (`entity_search_scoring`,
  `graph_persistence`, `skill_registration`, `tool_relevance`,
  `valkey_persistence`).

## Next Slice

- Continue with `tests/test_graph/tool_relevance.rs` and
  `tests/test_graph/graph_persistence.rs`.
