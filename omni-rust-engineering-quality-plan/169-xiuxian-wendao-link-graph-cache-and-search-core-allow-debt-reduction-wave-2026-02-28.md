# 169. Xiuxian-Wendao Link-Graph Cache and Search-Core Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph/cache_build.rs`
  - `tests/test_link_graph/search_core.rs`

## Why This Wave

After graph-lane convergence, the next highest-value LinkGraph slice was
`cache_build` plus `search_core`, which cover cache reusability, invalidation,
and primary retrieval payload behavior.

## Changes Implemented

Removed file-level `#![allow(...)]` from both scope files.

Root-cause cleanup after suppression removal:

- Replaced all `map_err(|e| e.to_string())` with `map_err(|e| e.clone())`
  across both files (`implicit_clone`).
- Replaced direct float equality with tolerance assertion in payload test:
  `abs(actual - expected) < 1e-12`.
- Updated assertion formatting to inline style in the reason diagnostic branch.
- Resolved follow-up compile/lint issues:
  - field-access format string (`{payload.reason}`) replaced by local binding.
  - removed unnecessary conversion (`f64::from(...)`) after type inference.

No suppression fallback was added.

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

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `39`
  - After this wave: `37`
  - Net reduction: `2` files

## Engineering Outcome

- LinkGraph cache and core retrieval tests now run suppression-free.
- Remaining LinkGraph suppression debt is concentrated in:
  - `tests/test_link_graph/build_scope.rs`
  - `tests/test_link_graph/graph_navigation.rs`
  - `tests/test_link_graph/markdown_attachments.rs`
  - `tests/test_link_graph/refresh.rs`
  - `tests/test_link_graph/search_match_strategies.rs`

## Next Slice

- Continue with:
  - `tests/test_link_graph/build_scope.rs`
  - `tests/test_link_graph/graph_navigation.rs`
