# 170. Xiuxian-Wendao Link-Graph Build-Scope and Navigation Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph/build_scope.rs`
  - `tests/test_link_graph/graph_navigation.rs`

## Why This Wave

After `cache_build` and `search_core`, the next LinkGraph slice was
scope-filtering and navigation diagnostics tests. These files are central to
directory filtering correctness and traversal telemetry semantics.

## Changes Implemented

Removed file-level `#![allow(...)]` from both scope files.

Root-cause cleanup:

- `build_scope.rs`
  - `map_err(|e| e.to_string())` -> `map_err(|e| e.clone())` (3 sites)
- `graph_navigation.rs`
  - `map_err(|e| e.to_string())` -> `map_err(|e| e.clone())` (3 sites)
  - Replaced float direct equality with tolerance assertions for:
    - `alpha`
    - `tol`
    - `partition_avg_node_count` (`8.0`, `3.0`)

No suppression fallback was introduced.

## Validation Evidence

1. Format + strict clippy:

```bash
cargo fmt -p xiuxian-wendao
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass

2. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `37`
  - After this wave: `35`
  - Net reduction: `2` files

## Engineering Outcome

- LinkGraph scope and navigation tests now run suppression-free.
- Remaining LinkGraph suppression debt is narrowed to:
  - `tests/test_link_graph/search_match_strategies.rs`
  - `tests/test_link_graph/refresh.rs`
  - `tests/test_link_graph/markdown_attachments.rs`

## Next Slice

- Continue with:
  - `tests/test_link_graph/search_match_strategies.rs`
  - `tests/test_link_graph/refresh.rs`
