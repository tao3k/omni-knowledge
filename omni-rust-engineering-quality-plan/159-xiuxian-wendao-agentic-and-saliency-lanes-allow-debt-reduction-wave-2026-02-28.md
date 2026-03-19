# 159. Xiuxian-Wendao Agentic and Saliency Lanes Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph_agentic/*`
  - `tests/test_link_graph_saliency/*`

## Why This Wave

After wave `158`, two concentrated suppression clusters remained in LinkGraph
agentic and saliency test lanes. Both lanes are central to Valkey-backed
runtime behavior and needed to converge without file-level lint suppression.

## Changes Implemented

1. Removed file-level `#![allow(...)]` in `test_link_graph_agentic`:
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_decide_promoted_with_audit.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_decide_rejects_invalid_transition.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_log_rejects_invalid_payload.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_log_roundtrip.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_log_trims_stream_by_max_entries.rs`

2. Removed file-level `#![allow(...)]` in `test_link_graph_saliency`:
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/compute_link_graph_saliency_activation_boosts_score.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/compute_link_graph_saliency_clamps_bounds.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_store_auto_repairs_invalid_payload.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_touch_and_get_with_valkey.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_touch_updates_inbound_edge_zset.rs`

3. Fixed newly exposed pedantic warnings without suppression:
   - `clippy::doc_markdown`: backticked `LinkGraph` in module docs
   - `clippy::implicit_clone`: replaced `.map_err(|err| err.to_string())?` with `.map_err(|err| err.clone())?`
   - `clippy::float_cmp`: replaced strict float equality with epsilon-based assertions
   - `clippy::manual_string_new`: replaced `\"\".to_string()` with `String::new()`
   - `clippy::cast_lossless`: replaced `idx as f64` with `f64::from(idx)`

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
- Summary: `286 passed`, `0 failed`, `1 skipped` (`12` slow tests)
- Nextest run ID: `52fb7078-c7e8-4492-839b-d9f368165f74`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `99`
  - After this wave: `87`
  - Net reduction: `12` files

## Engineering Outcome

- Agentic suggestion and saliency persistence lanes now run under strict pedantic
  clippy without file-level suppression.
- Remaining debt is concentrated in fewer files, improving follow-up targeting.

## Next Slice

- Continue with the remaining small/medium files under:
  - `tests/test_wendao_cli/*`
  - `tests/test_link_graph_ppr_benchmark/*`
  - `tests/test_dependency_*` and `tests/test_intent.rs`
