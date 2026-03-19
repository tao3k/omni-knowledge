# 316. xiuxian-qianji mechanism dispatch leaf routing module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler/mechanism_dispatch.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch.rs` (new)
  - `src/engine/compiler/mechanism_dispatch/stateless.rs`
  - `src/engine/compiler/mechanism_dispatch/stateful_cfg.rs`
  - `src/engine/compiler/task_type.rs`
- Goal: move non-stateful leaf-task routing out of dispatch shell and converge
  mechanism routing into three layers:
  - stateless paths,
  - cfg-sensitive stateful paths,
  - leaf task paths.

## Implementation

1. Added dedicated leaf dispatch module:
   - `compiler/mechanism_dispatch/leaf_dispatch.rs`
   - introduced:
     - `build(task_type, node_def) -> Result<Arc<dyn QianjiMechanism>, QianjiError>`
   - handles leaf tasks:
     - `calibration`, `mock`, `command`, `write_file`, `suspend`,
       `security_scan`, `wendao_ingester`, `wendao_refresh`, `router`.
2. Refactored parent dispatch shell:
   - `mechanism_dispatch.rs` now performs only:
     - task-type parsing
     - tiered delegation:
       - `stateless::build(...)`
       - `stateful_cfg::build(...)`
       - `leaf_dispatch::build(...)`
3. Updated submodule APIs:
   - `stateless.rs` now provides `build(task_type, compiler, node_def) -> Option<Arc<_>>`.
   - `stateful_cfg.rs` now provides
     `build(task_type, compiler, node_def) -> Option<Result<Arc<_>, QianjiError>>`.
4. `TaskType` ergonomics:
   - added `#[derive(Clone, Copy, Debug, Eq, PartialEq)]` in `task_type.rs` to
     support lightweight value routing across submodules.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateless.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch.rs`
  - result:
    - `mechanism_dispatch.rs`: `26`
    - `mechanism_dispatch/stateless.rs`: `23`
    - `mechanism_dispatch/stateful_cfg.rs`: `64`
    - `mechanism_dispatch/leaf_dispatch.rs`: `25`
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

- Dispatch shell complexity reduced further while preserving behavior.
- Leaf task routing now has a clear ownership boundary, reducing match-sprawl
  pressure in parent dispatch modules.
