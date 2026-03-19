# 329. xiuxian-qianji stateful dispatch and llm guard coverage (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `tests/test_compiler_dispatch_routes.rs`
- Goal: close remaining stateful dispatch coverage gaps by validating
  `formal_audit` native success and explicit `llm` task guard failure in
  non-`llm` builds.

## Implementation

1. Added stateful native success test:
   - `compiler_dispatches_formal_audit_native_path`
   - verifies `formal_audit` node compilation succeeds when no node-level LLM
     controller is declared.
2. Added non-feature guard failure test:
   - `compiler_rejects_llm_task_without_llm_feature`
   - verifies `task_type = "llm"` returns topology error requiring feature
     `'llm'` in non-feature builds.
3. Kept route-coverage concentration:
   - both additions live in the existing dispatch route suite
     `tests/test_compiler_dispatch_routes.rs`.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 skipped`, `0 failed`

## Outcome

- Stateful dispatch coverage now includes both native `formal_audit` positive
  path and non-feature `llm` task negative path.
- Compiler-route contract protection is now stronger across stateless, leaf,
  and stateful branches.
