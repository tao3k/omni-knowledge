# 173. Xiuxian-Wendao HMAS and KG-Cache Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_hmas.rs`
  - `tests/test_kg_cache.rs`

## Why This Wave

After LinkGraph convergence, the next low-coupling slice was top-level HMAS and
KG-cache tests. These files are moderately sized and expose contract-level
validation behavior without broad module fan-out.

## Changes Implemented

Removed file-level `#![allow(...)]` from both files.

Root-cause cleanup:

- `tests/test_kg_cache.rs`
  - `doc_markdown` fix in module docs:
    - wrapped `load_from_valkey_cached` and `invalidate` in backticks.
    - wrapped the example command in backticks.

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
  - Before this wave: `32`
  - After this wave: `30`
  - Net reduction: `2` files

## Engineering Outcome

- HMAS and KG-cache tests now run suppression-free under strict pedantic clippy.
- Remaining debt is concentrated in larger top-level and CLI/agentic scenario
  files.

## Next Slice

- Continue with small top-level files:
  - `tests/test_link_graph_evolution.rs`
  - `tests/test_dependency_debug.rs`
