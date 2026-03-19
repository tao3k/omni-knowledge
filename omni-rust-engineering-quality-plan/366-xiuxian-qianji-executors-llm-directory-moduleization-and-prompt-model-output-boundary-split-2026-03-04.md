# 366. xiuxian-qianji executors llm directory moduleization and prompt/model/output boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/executors/llm.rs`
  - added:
    - `src/executors/llm/mod.rs`
    - `src/executors/llm/mechanism.rs`
    - `src/executors/llm/model.rs`
    - `src/executors/llm/prompt.rs`
    - `src/executors/llm/output.rs`
- Goal:
  - replace mixed `llm` executor implementation with concern-split modules
    while preserving public `LlmAnalyzer` API and llm execution behavior.

## Implementation

1. Converted `executors::llm` to directory module:
   - `mod.rs` is interface-only and re-exports `LlmAnalyzer`.
2. Split responsibilities by concern:
   - `mechanism.rs`:
     - `LlmAnalyzer` model and mechanism execution flow.
   - `prompt.rs`:
     - semantic prompt-template resolution and context interpolation behavior.
   - `model.rs`:
     - model resolution policy (`llm_model` override and fallback logic).
   - `output.rs`:
     - JSON parsing from text/fenced blocks and repo-tree fallback output plan.
3. API compatibility preserved:
   - external path remains:
     `xiuxian_qianji::executors::llm::LlmAnalyzer`.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/executors/llm/mod.rs packages/rust/crates/xiuxian-qianji/src/executors/llm/mechanism.rs packages/rust/crates/xiuxian-qianji/src/executors/llm/model.rs packages/rust/crates/xiuxian-qianji/src/executors/llm/prompt.rs packages/rust/crates/xiuxian-qianji/src/executors/llm/output.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- LLM executor focused regression lanes:
  - `cargo nextest run -p xiuxian-qianji --features llm --test llm_analyzer --test llm_multi_tenancy`
  - result: `10 passed`, `0 failed`
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `llm` executor now follows interface-only module entry and clear boundaries
  for prompt assembly, model-selection policy, and response-output shaping.
- LLM and core compiler dispatch behavior remains stable under targeted lanes.
