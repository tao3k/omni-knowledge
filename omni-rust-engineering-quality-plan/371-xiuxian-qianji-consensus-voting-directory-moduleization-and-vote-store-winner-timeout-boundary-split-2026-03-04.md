# 371. xiuxian-qianji consensus voting directory moduleization and vote-store-winner-timeout boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed:
    - `src/consensus/manager/voting.rs`
  - added:
    - `src/consensus/manager/voting/mod.rs`
    - `src/consensus/manager/voting/submit.rs`
    - `src/consensus/manager/voting/vote_store.rs`
    - `src/consensus/manager/voting/winner.rs`
    - `src/consensus/manager/voting/timeout.rs`
- Goal:
  - replace mixed consensus-voting implementation with concern-split modules
    while preserving `ConsensusManager` public APIs and quorum behavior.

## Implementation

1. Converted `consensus::manager::voting` into a directory module:
   - `mod.rs` is interface-only and declares voting domain submodules.
2. Split voting concerns by domain:
   - `submit.rs`:
     - vote submission entrypoint (`submit_vote_payload`), winner-fast-path,
       threshold check, timeout decision.
   - `vote_store.rs`:
     - vote persistence (`HSET`), weight/agent counters, TTL refresh, output
       payload storage.
   - `winner.rs`:
     - winner read/set (`GET`/`SETNX`) and quorum publish path.
   - `timeout.rs`:
     - first-seen timeout guard and elapsed-time evaluation.
3. Applied strict visibility boundaries after split:
   - methods needed by parent `manager` module use
     `pub(in crate::consensus::manager)`.
   - cross-submodule helpers use `pub(super)`.
   - no visibility widened to public API surface.
4. Public API compatibility preserved:
   - external `ConsensusManager::{submit_vote, submit_vote_with_payload, wait_for_quorum, get_output_payload}`
     signatures unchanged.
5. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/consensus/manager/voting/mod.rs packages/rust/crates/xiuxian-qianji/src/consensus/manager/voting/submit.rs packages/rust/crates/xiuxian-qianji/src/consensus/manager/voting/vote_store.rs packages/rust/crates/xiuxian-qianji/src/consensus/manager/voting/winner.rs packages/rust/crates/xiuxian-qianji/src/consensus/manager/voting/timeout.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Consensus/swarm regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_consensus`
  - result: `3 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --test test_swarm_discovery --test test_swarm_orchestration`
  - result: `5 passed`, `0 failed` (`1 skipped`)
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `consensus::manager::voting` now follows interface-only entry and explicit
  boundary separation for submit flow, persistence/counters, winner lifecycle,
  and timeout checks.
- Quorum behavior and scheduler-facing consensus APIs remain stable under
  targeted and core qianji regression lanes.
