# 160. Xiuxian-Wendao Link-Graph PPR Benchmark Lane Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph_ppr_benchmark/mod.rs`
  - `tests/test_link_graph_ppr_benchmark/link_graph_related_ppr_latency_on_10k_fixture.rs`

## Why This Wave

After wave `159`, the LinkGraph PPR benchmark lane still relied on file-level
`#![allow(...)]`. This lane is important for performance observability and
needed to stay strict-clippy clean without blanket suppressions.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_benchmark/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_benchmark/link_graph_related_ppr_latency_on_10k_fixture.rs`

2. Fixed newly exposed lint issues in `mod.rs`:
   - `doc_markdown`: backticked `LinkGraph` in module docs
   - `format_push_string`: replaced `push_str(format!(...))` with direct `push_str` composition
   - cast-related warnings in percentile helper:
     - replaced floating percentile API with integer percentile (`u32`) API
     - removed float-to-int cast path for rank calculation
     - updated call sites in benchmark test from `0.50/0.95` to `50/95`

3. Kept benchmark behavior intact:
   - fixture generation unchanged semantically
   - runtime diagnostics and budget assertions unchanged semantically
   - heavy benchmark remains `#[ignore]` by default

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
- Nextest run ID: `f4dd7d49-1fae-4d37-80b6-fc0e7bbc4516`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `87`
  - After this wave: `85`
  - Net reduction: `2` files

## Engineering Outcome

- PPR benchmark lane now runs suppression-free under strict pedantic clippy.
- Percentile helper now has a clearer integer-based contract and avoids
  fragile cast behavior.

## Next Slice

- Continue with `tests/test_wendao_cli/related/*` and
  `tests/test_wendao_cli/search/*` leaf files where suppressions remain
  concentrated and file sizes are moderate.
