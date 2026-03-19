# 171. Xiuxian-Wendao Link-Graph Match-Strategy and Refresh Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph/search_match_strategies.rs`
  - `tests/test_link_graph/refresh.rs`

## Why This Wave

With scope/navigation converged, the next LinkGraph slice was matching-strategy
semantics and incremental refresh behavior, both of which are heavily exercised
in runtime search and sync pathways.

## Changes Implemented

Removed file-level `#![allow(...)]` from both scope files.

Root-cause cleanup:

- Replaced all `map_err(|e| e.to_string())` with `map_err(|e| e.clone())`
  across both files (`implicit_clone`).

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
  - Before this wave: `35`
  - After this wave: `33`
  - Net reduction: `2` files

## Engineering Outcome

- LinkGraph suppression debt is now narrowed to a single file:
  `tests/test_link_graph/markdown_attachments.rs`.
- This keeps the lane close to full suppression-free convergence.

## Next Slice

- Continue with:
  - `tests/test_link_graph/markdown_attachments.rs`
