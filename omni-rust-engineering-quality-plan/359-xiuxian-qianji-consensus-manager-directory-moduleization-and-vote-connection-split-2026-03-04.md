# 359. xiuxian-qianji consensus manager directory moduleization and vote/connection split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/consensus/manager.rs`
  - added:
    - `src/consensus/manager/mod.rs`
    - `src/consensus/manager/keys.rs`
    - `src/consensus/manager/time.rs`
    - `src/consensus/manager/connection.rs`
    - `src/consensus/manager/voting.rs`
- Goal:
  - replace monolithic consensus-manager implementation with concern-split
    modules while preserving public `ConsensusManager` APIs and distributed
    vote behavior.

## Implementation

1. Converted `consensus::manager` to directory module:
   - `manager/mod.rs` keeps public entrypoint methods:
     - constructors (`new`, `with_agent_identity`)
     - vote submission APIs
     - payload lookup API
     - quorum wait API
2. Split internal concerns:
   - `keys.rs`:
     - vote keyspace descriptor (`VoteKeys`) and vote snapshot model.
   - `time.rs`:
     - timestamp helper for vote events.
   - `connection.rs`:
     - Valkey command execution, reconnect, and connection invalidation flow.
   - `voting.rs`:
     - vote record path, threshold evaluation, winner publication,
       timeout detection, and output payload persistence.
3. Visibility hardening:
   - used `pub(super)` selectively for cross-file internal helpers.
   - kept public API surface unchanged at `ConsensusManager`.
4. No broad lint suppressions added.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/consensus/manager/mod.rs packages/rust/crates/xiuxian-qianji/src/consensus/manager/keys.rs packages/rust/crates/xiuxian-qianji/src/consensus/manager/time.rs packages/rust/crates/xiuxian-qianji/src/consensus/manager/connection.rs packages/rust/crates/xiuxian-qianji/src/consensus/manager/voting.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_consensus`
  - result: `3 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Consensus runtime now follows interface-only module entry and concern-based
  submodule boundaries.
- Vote path, connection path, and quorum wait path are easier to evolve and
  audit independently with behavior preserved.
