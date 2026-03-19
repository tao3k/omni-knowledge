# 358. xiuxian-qianji bootcamp directory moduleization and workflow/runtime/llm split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/bootcamp.rs`
  - added:
    - `src/bootcamp/mod.rs`
    - `src/bootcamp/model.rs`
    - `src/bootcamp/workflow.rs`
    - `src/bootcamp/manifest.rs`
    - `src/bootcamp/runtime.rs`
    - `src/bootcamp/llm.rs`
- Goal:
  - replace monolithic bootcamp orchestration module with focused domain
    modules while preserving all public bootcamp APIs and behavior.

## Implementation

1. Converted `bootcamp` into a directory module:
   - `bootcamp/mod.rs` is now interface-only and re-exports:
     - `WorkflowReport`
     - `BootcampLlmMode`
     - `BootcampRunOptions`
     - `BootcampVfsMount`
     - `run_workflow`
     - `run_workflow_with_mounts`
     - `run_scenario`
2. Split responsibilities by concern:
   - `model.rs`:
     - report/options/mount data structures and defaults.
     - feature-gated `BootcampLlmMode` variants.
   - `workflow.rs`:
     - run entrypoints and scheduler wiring.
   - `manifest.rs`:
     - URI-to-manifest resolution and manifest-level LLM requirement detection.
   - `runtime.rs`:
     - repo-root resolution, LinkGraph index bootstrap, time helpers.
   - `llm.rs`:
     - feature-gated bootcamp LLM client resolution and mock client.
3. Kept API compatibility:
   - no public signature changes for bootcamp exports consumed via
     `xiuxian_qianji::{...}`.
4. No broad lint suppression added.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/bootcamp/mod.rs packages/rust/crates/xiuxian-qianji/src/bootcamp/model.rs packages/rust/crates/xiuxian-qianji/src/bootcamp/manifest.rs packages/rust/crates/xiuxian-qianji/src/bootcamp/runtime.rs packages/rust/crates/xiuxian-qianji/src/bootcamp/llm.rs packages/rust/crates/xiuxian-qianji/src/bootcamp/workflow.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_bootcamp_api`
  - result: `3 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `bootcamp` now follows the repository modularization standard (interface-only
  module entrypoint + concern-based child modules).
- Internal code navigation and future extension points are clearer without
  changing external behavior.
