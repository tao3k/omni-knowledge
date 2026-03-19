# 318. xiuxian-qianji leaf dispatch directory module split (`io_control`, `quality_guard`, `wendao_router`) (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch.rs` (removed)
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/mod.rs` (new)
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/io_control.rs` (new)
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/quality_guard.rs` (new)
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/wendao_router.rs` (new)
- Goal: split leaf-task routing by domain responsibility so command/control,
  safety/quality, and wendao/router paths are no longer mixed in one file.

## Implementation

1. Converted `leaf_dispatch.rs` to directory module:
   - removed single-file implementation.
   - added `leaf_dispatch/mod.rs` as routing shell.
2. Split leaf routing into focused domain modules:
   - `io_control.rs`:
     - `command`, `write_file`, `suspend`
   - `quality_guard.rs`:
     - `calibration`, `mock`, `security_scan`
   - `wendao_router.rs`:
     - `wendao_ingester`, `wendao_refresh`, `router`
3. Preserved parent contract:
   - `leaf_dispatch::build(task_type, node_def)` still returns
     `Result<Arc<dyn QianjiMechanism>, QianjiError>`.
   - internal fallback error for impossible task mismatch remains.
4. No behavior changes:
   - only module boundaries and delegation paths were adjusted.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/io_control.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/quality_guard.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/wendao_router.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg/formal_audit.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg/llm.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateless.rs`
  - result:
    - `mechanism_dispatch.rs`: `26`
    - `leaf_dispatch/mod.rs`: `30`
    - `leaf_dispatch/io_control.rs`: `16`
    - `leaf_dispatch/quality_guard.rs`: `16`
    - `leaf_dispatch/wendao_router.rs`: `17`
    - `stateful_cfg/mod.rs`: `21`
    - `stateful_cfg/formal_audit.rs`: `33`
    - `stateful_cfg/llm.rs`: `28`
    - `stateless.rs`: `23`
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

- Leaf routing is now explicitly layered by task domain, which improves
  readability and reduces future merge pressure in a single leaf dispatch file.
- `mod.rs` remains interface-only orchestration, aligned with directory-module
  standards.
