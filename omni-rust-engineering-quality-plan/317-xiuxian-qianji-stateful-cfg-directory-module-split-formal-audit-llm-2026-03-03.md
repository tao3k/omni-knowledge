# 317. xiuxian-qianji `stateful_cfg` directory module split (`formal_audit` + `llm`) (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler/mechanism_dispatch/stateful_cfg.rs` (removed)
  - `src/engine/compiler/mechanism_dispatch/stateful_cfg/mod.rs` (new)
  - `src/engine/compiler/mechanism_dispatch/stateful_cfg/formal_audit.rs` (new)
  - `src/engine/compiler/mechanism_dispatch/stateful_cfg/llm.rs` (new)
- Goal: split cfg-sensitive stateful routing by domain so `formal_audit` and
  `llm` do not share one mixed file.

## Implementation

1. Converted single-file `stateful_cfg.rs` into a directory module:
   - removed `stateful_cfg.rs`.
   - added `stateful_cfg/mod.rs` as interface-only router.
2. Extracted domain files:
   - `stateful_cfg/formal_audit.rs`
     - keeps LLM-controller-aware formal-audit routing.
     - preserves non-`llm` feature guard behavior.
   - `stateful_cfg/llm.rs`
     - keeps node/global client resolution path and llm mechanism construction.
     - preserves non-`llm` feature error path.
3. Preserved parent dispatch contract:
   - `stateful_cfg::build(task_type, compiler, node_def)` signature unchanged.
4. Kept warning-free imports:
   - cfg-gated imports adjusted to avoid unused-import warnings in non-`llm`
     builds.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateless.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg/formal_audit.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg/llm.rs`
  - result:
    - `mechanism_dispatch.rs`: `26`
    - `leaf_dispatch.rs`: `25`
    - `stateless.rs`: `23`
    - `stateful_cfg/mod.rs`: `21`
    - `stateful_cfg/formal_audit.rs`: `33`
    - `stateful_cfg/llm.rs`: `28`
- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- cfg-sensitive routing is now split by behavior domain, reducing mixed-branch
  cognitive load in a single file.
- `stateful_cfg/mod.rs` stays interface-only, aligned with module boundary
  standards.
