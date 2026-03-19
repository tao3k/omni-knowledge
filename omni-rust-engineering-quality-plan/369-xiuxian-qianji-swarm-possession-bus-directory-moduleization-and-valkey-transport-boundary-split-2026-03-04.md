# 369. xiuxian-qianji swarm possession bus directory moduleization and valkey transport boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed:
    - `src/swarm/possession/bus.rs`
  - added:
    - `src/swarm/possession/bus/mod.rs`
    - `src/swarm/possession/bus/keys.rs`
    - `src/swarm/possession/bus/connection.rs`
    - `src/swarm/possession/bus/request.rs`
    - `src/swarm/possession/bus/response.rs`
- Goal:
  - replace mixed remote-possession transport logic with concern-split modules
    while preserving public `RemotePossessionBus` API and Valkey protocol
    behavior.

## Implementation

1. Converted `swarm::possession::bus` into a directory module:
   - `mod.rs` now owns struct construction and module boundaries only.
2. Split Valkey transport concerns by domain:
   - `keys.rs`:
     - request/response storage key and queue/channel key construction.
   - `connection.rs`:
     - connection acquisition, reconnect invalidation, and shared `run_command`
       retry path.
   - `request.rs`:
     - request submit, queue claim, and request+wait orchestration.
   - `response.rs`:
     - response submit, publish-notify, and pubsub wait path.
3. API compatibility preserved:
   - external path remains:
     `xiuxian_qianji::swarm::RemotePossessionBus`.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/swarm/possession/bus/mod.rs packages/rust/crates/xiuxian-qianji/src/swarm/possession/bus/keys.rs packages/rust/crates/xiuxian-qianji/src/swarm/possession/bus/connection.rs packages/rust/crates/xiuxian-qianji/src/swarm/possession/bus/request.rs packages/rust/crates/xiuxian-qianji/src/swarm/possession/bus/response.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Swarm transport regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_swarm_discovery`
  - result: `2 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --test test_swarm_orchestration`
  - result: `3 passed`, `0 failed` (`1 skipped`)
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `swarm::possession::bus` now follows interface-only module entry with clear
  boundaries for key modeling, connection lifecycle, request orchestration, and
  response wait/publish flow.
- Existing swarm protocol behavior remains stable across both swarm-specific and
  core compiler dispatch validation lanes.
