# 375. xiuxian-qianji scheduler consensus resolve directory moduleization and call-handler-policy boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed:
    - `src/scheduler/core/consensus/resolve.rs`
  - added:
    - `src/scheduler/core/consensus/resolve/mod.rs`
    - `src/scheduler/core/consensus/resolve/call_ctx.rs`
    - `src/scheduler/core/consensus/resolve/handlers.rs`
    - `src/scheduler/core/consensus/resolve/policy.rs`
- Goal:
  - replace mixed consensus-resolution logic with concern-split modules while
    preserving scheduler consensus behavior and telemetry outcomes.

## Implementation

1. Converted `scheduler::core::consensus::resolve` to a directory module:
   - `mod.rs` retains top-level resolve orchestration (`resolve_consensus_output`).
2. Split consensus resolution concerns by domain:
   - `call_ctx.rs`:
     - shared call context structure for manager/session/hash/output bindings.
   - `handlers.rs`:
     - `handle_consensus_agreed` and `handle_consensus_pending` flows
       (checkpoint/status/update + wait-for-quorum orchestration).
   - `policy.rs`:
     - `consensus_target_progress` telemetry target derivation.
3. Performed post-split warning cleanup:
   - removed stale imports from `mod.rs` to keep `check/clippy` warning-clean.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/scheduler/core/consensus/resolve/mod.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/consensus/resolve/call_ctx.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/consensus/resolve/handlers.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/consensus/resolve/policy.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Consensus/scheduler regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_consensus --test test_scheduler_affinity_failover`
  - result: `5 passed`, `0 failed`
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Consensus resolution now follows interface-only `mod.rs` entry with explicit
  boundaries for shared call context, branch handlers, and telemetry policy
  calculation.
- Scheduler consensus decision behavior remains stable under targeted and core
  qianji regression lanes.
