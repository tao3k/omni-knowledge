# 172. Xiuxian-Wendao Link-Graph Markdown-Attachments Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph/markdown_attachments.rs`

## Why This Wave

This was the final remaining suppression file inside `tests/test_link_graph`.
Closing it completes suppression-free convergence for the full LinkGraph test
directory.

## Changes Implemented

Removed file-level `#![allow(...)]` from
`tests/test_link_graph/markdown_attachments.rs`.

Root-cause cleanup:

- Replaced all `map_err(|e| e.to_string())` with `map_err(|e| e.clone())`
  (`implicit_clone`) at 5 call sites.

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
  - Before this wave: `33`
  - After this wave: `32`
  - Net reduction: `1` file

## Engineering Outcome

- `tests/test_link_graph/*` is now fully suppression-free.
- Remaining suppression debt is outside LinkGraph, concentrated in top-level and
  CLI/agentic integration-heavy test files.

## Next Slice

- Continue with top-level small files:
  - `tests/test_hmas.rs`
  - `tests/test_kg_cache.rs`
