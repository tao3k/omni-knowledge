# 370. xiuxian-qianji swarm discovery registry directory moduleization and connection-heartbeat-discovery boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed:
    - `src/swarm/discovery/registry.rs`
  - added:
    - `src/swarm/discovery/registry/mod.rs`
    - `src/swarm/discovery/registry/keys.rs`
    - `src/swarm/discovery/registry/payload.rs`
    - `src/swarm/discovery/registry/connection.rs`
    - `src/swarm/discovery/registry/heartbeat.rs`
    - `src/swarm/discovery/registry/discover.rs`
- Goal:
  - replace mixed discovery-registry implementation with concern-split modules
    while preserving public `GlobalSwarmRegistry` API and heartbeat/discovery
    behavior.

## Implementation

1. Converted `swarm::discovery::registry` into a directory module:
   - `mod.rs` now owns struct declaration and constructor only.
2. Split discovery-registry concerns by domain:
   - `keys.rs`:
     - node key construction for registry entries.
   - `payload.rs`:
     - heartbeat payload shaping and input validation.
   - `connection.rs`:
     - Valkey connection lifecycle and retrying command dispatch.
   - `heartbeat.rs`:
     - heartbeat write path and background loop spawning.
   - `discover.rs`:
     - discover-all/by-role, candidate selection, stale-index pruning.
3. API compatibility preserved:
   - external path remains:
     `xiuxian_qianji::swarm::GlobalSwarmRegistry`.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/swarm/discovery/registry/mod.rs packages/rust/crates/xiuxian-qianji/src/swarm/discovery/registry/keys.rs packages/rust/crates/xiuxian-qianji/src/swarm/discovery/registry/payload.rs packages/rust/crates/xiuxian-qianji/src/swarm/discovery/registry/connection.rs packages/rust/crates/xiuxian-qianji/src/swarm/discovery/registry/heartbeat.rs packages/rust/crates/xiuxian-qianji/src/swarm/discovery/registry/discover.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Swarm regression lanes:
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

- `swarm::discovery::registry` now follows interface-only module entry with
  explicit boundaries for payload modeling, Valkey connection management,
  heartbeat lifecycle, and discovery/query behavior.
- Registry behavior remains stable across swarm-specific and core compiler
  dispatch validation lanes.
