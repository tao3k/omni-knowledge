# 324. xiuxian-qianji compiler dispatch routes integration tests (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `tests/test_compiler_dispatch_routes.rs` (new)
- Goal: lock down resolver-chain dispatch behavior at the compiler boundary
  with route-focused integration coverage across stateless, leaf, and topology
  guard paths.

## Implementation

1. Added a new top-level integration test file:
   - `tests/test_compiler_dispatch_routes.rs`
2. Added a shared compiler-construction helper for test isolation:
   - builds a temporary `LinkGraphIndex`
   - instantiates `QianjiCompiler` with built-in `ThousandFacesOrchestrator`
     and `PersonaRegistry`
3. Added stateless lane coverage:
   - `compiler_dispatches_knowledge_task_via_stateless_lane`
   - verifies a `knowledge` manifest compiles successfully.
4. Added leaf lane coverage:
   - `compiler_dispatches_command_task_via_leaf_lane`
   - verifies a `command` manifest compiles successfully.
5. Added non-`llm` stateful guard coverage:
   - `compiler_rejects_llm_augmented_formal_audit_without_llm_feature`
   - verifies `formal_audit` + `[nodes.qianhuan]` + `[nodes.llm]` fails with
     feature-gate guidance when built without `llm`.
6. Added unknown-task topology error coverage:
   - `compiler_rejects_unknown_task_type_with_topology_error`
   - verifies unknown `task_type` surfaces the expected topology error text.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `6 passed`, `0 skipped`, `0 failed`

## Outcome

- Compiler dispatch routes now have explicit integration-level lock tests for
  key resolver lanes and error contracts.
- Refactoring risk is reduced for future resolver-chain changes because route
  behavior is now tested via externally visible compile outcomes.
