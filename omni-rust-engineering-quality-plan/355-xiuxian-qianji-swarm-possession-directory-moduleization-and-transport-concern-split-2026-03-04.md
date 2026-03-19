# 355. xiuxian-qianji swarm possession directory moduleization and transport concern split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/swarm/possession.rs`
  - added:
    - `src/swarm/possession/mod.rs`
    - `src/swarm/possession/model.rs`
    - `src/swarm/possession/bus.rs`
    - `src/swarm/possession/error_map.rs`
    - `src/swarm/possession/util.rs`
- Goal:
  - split monolithic remote-possession implementation into domain modules while
    preserving public API:
    - `RemoteNodeRequest`
    - `RemoteNodeResponse`
    - `RemotePossessionBus`
    - `map_execution_error_to_response`

## Implementation

1. Converted `swarm::possession` into directory module:
   - `mod.rs` now acts as interface-only export entrypoint.
2. Split concerns by domain:
   - `model.rs`:
     - request/response protocol models
     - request-id/timestamp generation
     - success/failure response constructors.
   - `bus.rs`:
     - Valkey transport orchestration (`submit_request`, `claim_next_for_role`,
       `submit_response`, `wait_response`, `request_and_wait`)
     - reconnect-aware command execution boundary.
   - `error_map.rs`:
     - centralized conversion from execution error to failed remote response.
   - `util.rs`:
     - unix-millis time helper reused by model and error mapping.
3. Preserved behavior:
   - Valkey key naming and queue/channel semantics unchanged
   - request/response payload schemas unchanged
   - pubsub wait and timeout behavior unchanged
   - reconnect/retry command path unchanged.
4. Lint hygiene:
   - removed one unused import surfaced by clippy (`HashMap` in `bus.rs`).
   - no broad lint suppression introduced.

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

- Remote possession now follows modularization standards with clear protocol vs
  transport vs error-mapping boundaries.
- Public API remains stable and key qianji regression lanes stay green.
