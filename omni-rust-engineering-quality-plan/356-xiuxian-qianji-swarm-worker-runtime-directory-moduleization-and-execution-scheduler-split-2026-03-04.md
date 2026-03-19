# 356. xiuxian-qianji swarm worker runtime directory moduleization and execution/scheduler split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/swarm/engine/worker/runtime.rs`
  - added:
    - `src/swarm/engine/worker/runtime/mod.rs`
    - `src/swarm/engine/worker/runtime/execution.rs`
    - `src/swarm/engine/worker/runtime/scheduler.rs`
    - `src/swarm/engine/worker/runtime/reporting.rs`
    - `src/swarm/engine/worker/runtime/telemetry.rs`
- Goal:
  - split monolithic worker runtime implementation into focused modules while
    preserving `SwarmEngine` runtime behavior and orchestration contracts.

## Implementation

1. Converted `worker::runtime` into directory module:
   - `runtime/mod.rs` now declares runtime submodules only.
2. Split concerns by domain:
   - `execution.rs`:
     - worker task spawn entrypoint
     - worker run loop orchestration and cancellation handling.
   - `scheduler.rs`:
     - runtime scheduler assembly
     - consensus/role-registry/remote-possession service wiring.
   - `reporting.rs`:
     - worker result shaping into `SwarmAgentReport`
     - window stats extraction and failure propagation.
   - `telemetry.rs`:
     - non-blocking pulse emission helper.
3. Preserved behavior:
   - remote responder start/stop lifecycle unchanged
   - checkpoint run + global cancellation race unchanged
   - scheduler runtime service composition unchanged
   - telemetry semantics unchanged.
4. No broad lint suppression introduced.

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

- Worker runtime now follows modularization standards with clear separation of
  execution loop, scheduler wiring, reporting, and telemetry concerns.
- Core qianji execution and swarm regression lanes remain green.
