# 168. Xiuxian-Wendao Graph Lane Core Files Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_graph/entity_search_scoring.rs`
  - `tests/test_graph/graph_persistence.rs`
  - `tests/test_graph/skill_registration.rs`
  - `tests/test_graph/tool_relevance.rs`
  - `tests/test_graph/valkey_persistence.rs`

## Why This Wave

After wave `167`, the remaining `test_graph` suppression debt was concentrated
in five core files that drive graph ranking, persistence, registration, and
tool relevance behavior.

## Changes Implemented

Removed file-level `#![allow(...)]` from all five scope files.

Root-cause fixes exposed by strict clippy:

- `entity_search_scoring.rs`
  - `format!("tool:{}", name)` -> `format!("tool:{name}")` (2 sites)
- `graph_persistence.rs`
  - `format!("Description of {}", name)` -> `format!("Description of {name}")`
  - `format!("{} -> {}", source, target)` -> `format!("{source} -> {target}")`
- `skill_registration.rs`
  - Inlined assertion formatting: `"... got: {names:?}"`
- `tool_relevance.rs`
  - Inlined assertion formatting with `{tool_names:?}` and `{cs}`/`{ss}`
- `valkey_persistence.rs`
  - Replaced float direct equality with tolerance checks:
    `abs(actual - expected) < 1e-9`

No suppression fallback was introduced.

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

- Final result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

Note: an earlier run in restricted mode showed `Operation not permitted` in
`test_link_graph::cache_build::*`; rerun in full-access environment passed,
indicating environment constraint rather than code regression.

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `44`
  - After this wave: `39`
  - Net reduction: `5` files

## Engineering Outcome

- `test_graph` lane is now fully suppression-free.
- Remaining suppression debt is concentrated in broader integration lanes:
  `test_link_graph*`, `test_wendao_cli/agentic*`, and top-level stress/indexer
  suites.

## Next Slice

- Continue with `tests/test_link_graph/cache_build.rs` and
  `tests/test_link_graph/search_core.rs`.
