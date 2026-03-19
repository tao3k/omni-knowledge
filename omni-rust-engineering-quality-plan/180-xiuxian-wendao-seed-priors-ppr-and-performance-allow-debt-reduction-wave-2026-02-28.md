# 180. Xiuxian-Wendao Seed-Priors, PPR, and Performance Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph_seed_and_priors/mod.rs`
  - `tests/test_link_graph_ppr_weighting.rs`
  - `tests/test_performance_stress.rs`

## Why This Wave

These three files are compact and independent enough for low-risk,
high-throughput suppression-debt reduction.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_link_graph_seed_and_priors/mod.rs`
   - `tests/test_link_graph_ppr_weighting.rs`
   - `tests/test_performance_stress.rs`

2. Root-cause cleanup:
   - `test_performance_stress.rs`
     - `cast_lossless` fix: `i as f64` -> `f64::from(i)`
   - `test_link_graph_seed_and_priors/mod.rs` and
     `test_link_graph_ppr_weighting.rs`
     - removal was clean without additional source changes

No suppressions were reintroduced.

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
- Note: one intermediate `cast_lossless` warning was fixed and rerun to clean
  pass.

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `20`
  - After this wave: `17`
  - Net reduction: `3` files

## Engineering Outcome

- Seed-prior integration entrypoint, weighted-PPR behavior check, and
  narrator performance stress lane are now suppression-free.
- Remaining debt is concentrated in larger CLI/agentic scenario suites and
  seed-priors scenario leaves.
