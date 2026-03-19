# 357. xiuxian-qianji swarm engine types directory moduleization and model boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/swarm/engine/types.rs`
  - added:
    - `src/swarm/engine/types/mod.rs`
    - `src/swarm/engine/types/runtime.rs`
    - `src/swarm/engine/types/agent.rs`
    - `src/swarm/engine/types/options.rs`
    - `src/swarm/engine/types/report.rs`
- Goal:
  - split mixed responsibility `types.rs` into focused model/config/runtime
    modules while preserving all existing `swarm::engine` public type APIs.

## Implementation

1. Converted `swarm::engine::types` into directory module:
   - `types/mod.rs` now owns re-export boundary for:
     - `SwarmAgentConfig`
     - `SwarmExecutionOptions`
     - `SwarmAgentReport`
     - `SwarmExecutionReport`
2. Split concerns by domain:
   - `runtime.rs`:
     - internal worker runtime types (`WorkerJoinSet`, `WorkerRuntimeConfig`)
     - session-id generation helper.
   - `agent.rs`:
     - `SwarmAgentConfig` model and constructor defaults.
   - `options.rs`:
     - `SwarmExecutionOptions` model and default policy values.
   - `report.rs`:
     - `SwarmAgentReport` and `SwarmExecutionReport` result models.
3. Visibility boundary cleanup:
   - set runtime-internal visibility to `pub(in crate::swarm::engine)` so
     sibling modules can consume internal types without overexposing them.
   - added explicit type annotations in worker scheduler wiring to keep
     inference stable after the split.
4. No behavior changes intended and no lint suppressions introduced.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Existing qianji dispatch regressions:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`
- Swarm discovery lane recheck (network/socket dependent):
  - sandbox run failed with OS-level permission (`Operation not permitted`)
  - rerun with elevated execution:
    - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_swarm_discovery`
    - result: `2 passed`, `0 failed`

## Outcome

- Swarm engine types now follow modularization standards with explicit runtime
  internal boundaries and cleaner model grouping.
- Public APIs remain stable and qianji regression lanes remain green.
