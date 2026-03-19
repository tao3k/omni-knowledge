# 377. xiuxian-qianji scheduler types directory moduleization and runtime-bundle boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed:
    - `src/scheduler/core/types.rs`
  - added:
    - `src/scheduler/core/types/mod.rs`
    - `src/scheduler/core/types/constants.rs`
    - `src/scheduler/core/types/consensus.rs`
    - `src/scheduler/core/types/remote.rs`
    - `src/scheduler/core/types/services.rs`
    - `src/scheduler/core/types/scheduler.rs`
    - `src/scheduler/core/types/constructors.rs`
- Goal:
  - replace mixed scheduler core types implementation with concern-split modules
    while preserving scheduler public API and internal runtime wiring.

## Implementation

1. Converted `scheduler::core::types` into a directory module:
   - `mod.rs` now serves as interface-only re-export surface.
2. Split responsibilities by domain:
   - `constants.rs`:
     - wait/timeout/TTL constants for consensus and remote delegation paths.
   - `consensus.rs`:
     - checkpoint snapshot view and consensus outcome model.
   - `remote.rs`:
     - remote delegation outcome model.
   - `services.rs`:
     - `SchedulerRuntimeServices` runtime dependency bundle.
   - `scheduler.rs`:
     - `QianjiScheduler` struct and runtime fields.
   - `constructors.rs`:
     - scheduler constructor family (`new`, `with_consensus_manager`, etc.).
3. Visibility boundary correction after split:
   - promoted internal shared items/fields from `pub(super)` to
     `pub(in crate::scheduler::core)` to preserve sibling-module access in
     `core/*` runtime lanes.
4. Public compatibility preserved:
   - `scheduler::core::{QianjiScheduler, SchedulerRuntimeServices}` exports unchanged.
5. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/scheduler/core/types/mod.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/types/constants.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/types/consensus.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/types/remote.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/types/services.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/types/scheduler.rs packages/rust/crates/xiuxian-qianji/src/scheduler/core/types/constructors.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Scheduler runtime regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_scheduler_affinity_failover --test test_consensus --test test_swarm_orchestration`
  - result: `8 passed`, `0 failed` (`1 skipped`)
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
- Feature lane note:
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: blocked by unrelated compile errors in
    `packages/rust/crates/xiuxian-llm/src/llm/providers/openai_like.rs`
    (`LiteChatRequest`/`LlmResult`/`ImageSource` unresolved).

## Outcome

- `scheduler::core::types` now follows interface-only module entry and clear
  separation between runtime constants/models/service bundle/scheduler struct
  and constructors.
- Non-LLM scheduler and core qianji lanes remain stable; LLM lane is currently
  externally blocked by ongoing changes in `xiuxian-llm`.
