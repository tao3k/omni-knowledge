# 354. xiuxian-qianji swarm discovery directory moduleization and registry concern split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/swarm/discovery.rs`
  - added:
    - `src/swarm/discovery/mod.rs`
    - `src/swarm/discovery/model.rs`
    - `src/swarm/discovery/parse.rs`
    - `src/swarm/discovery/registry.rs`
    - `src/swarm/discovery/util.rs`
- Goal:
  - replace monolithic swarm discovery implementation with domain-focused
    modules while preserving public API:
    - `ClusterNodeIdentity`
    - `ClusterNodeRecord`
    - `GlobalSwarmRegistry`

## Implementation

1. Converted `swarm::discovery` into a directory module:
   - `mod.rs` now serves as interface-only export boundary.
2. Split concerns by domain:
   - `model.rs`:
     - identity/record data models
     - identity sanitization.
   - `registry.rs`:
     - `GlobalSwarmRegistry` state
     - heartbeat/discovery/pick-candidate APIs
     - command execution and reconnect logic.
   - `parse.rs`:
     - record parsing from hash fields
     - role filter matching.
   - `util.rs`:
     - optional-text normalization
     - unix-millis timestamp helper
     - registry index key constant.
3. Preserved behavior:
   - heartbeat payload structure unchanged
   - stale-index pruning behavior unchanged
   - role-filter and candidate-picking semantics unchanged
   - Valkey reconnect retry path unchanged
4. No broad lint suppression introduced.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted swarm discovery regression:
  - initial sandbox run failed with OS-level permission (`Operation not permitted`).
  - rerun outside sandbox restrictions:
    - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_swarm_discovery`
    - result: `2 passed`, `0 failed`
- Existing qianji dispatch regressions:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Swarm discovery now follows modularization standards with clear ownership
  boundaries (`model`/`registry`/`parse`/`util`).
- Public API and runtime behavior remain stable and test-validated.
