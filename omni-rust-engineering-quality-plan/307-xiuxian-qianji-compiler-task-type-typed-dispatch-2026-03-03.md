# 307. xiuxian-qianji compiler task type typed dispatch (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/task_type.rs` (new)
- Goal: replace string-literal task dispatch in compiler core with a typed
  task-kind parser to centralize unknown-task validation and reduce dispatch noise.

## Implementation

1. Added typed task-kind module:
   - `src/engine/compiler/task_type.rs`
   - introduced:
     - `TaskType` enum
     - `TaskType::parse(raw)` parser with unified unknown-task error contract
2. Updated compiler dispatch:
   - `compiler.rs` now declares `mod task_type;`
   - `build_mechanism` now parses `node_def.task_type` via
     `task_type::TaskType::parse(...)` and matches on typed variants.
3. Preserved behavior:
   - task-to-mechanism mapping remains unchanged
   - cfg-gated `formal_audit`/`llm` behavior remains unchanged
   - unknown task error remains `Unknown task type: <raw>`

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_type.rs`
  - result:
    - `compiler.rs`: `370`
    - `compiler/task_type.rs`: `38`
- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- Compiler dispatch now uses a typed task model instead of scattered string comparisons.
- Unknown-task validation is centralized and easier to maintain as task surface evolves.
