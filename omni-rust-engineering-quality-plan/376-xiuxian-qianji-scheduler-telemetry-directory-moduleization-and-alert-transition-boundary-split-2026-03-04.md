# 376. xiuxian-qianji scheduler telemetry directory moduleization and alert-transition boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed:
    - `src/scheduler/core/telemetry.rs`
  - added:
    - `src/scheduler/core/telemetry/mod.rs`
    - `src/scheduler/core/telemetry/node_transition.rs`
    - `src/scheduler/core/telemetry/alerts.rs`
- Goal:
  - replace mixed scheduler telemetry implementation with concern-split modules
    while preserving non-blocking emission behavior and scheduler call-site
    contracts.

## Implementation

1. Converted `scheduler::core::telemetry` into a directory module:
   - `mod.rs` now owns shared non-blocking event emission (`emit_event_non_blocking`).
2. Split scheduler telemetry concerns by domain:
   - `node_transition.rs`:
     - node transition event materialization (`emit_node_transition`).
   - `alerts.rs`:
     - consensus spike and affinity alert emission
       (`emit_consensus_spike`, `emit_affinity_alert`).
3. API compatibility preserved:
   - scheduler internal method names/signatures remain unchanged, so run-loop
     and consensus call-sites required no behavior-level rewrites.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/scheduler/core/telemetry/mod.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/telemetry/node_transition.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/telemetry/alerts.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Telemetry/scheduler regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_swarm_orchestration --test test_scheduler_affinity_failover`
  - result: `5 passed`, `0 failed` (`1 skipped`)
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Scheduler telemetry now follows interface-only `mod.rs` with explicit
  boundary separation between emission runtime, node transition events, and
  alert/spike event builders.
- Existing scheduler/swarm telemetry behavior remains stable under targeted and
  core qianji regression lanes.
