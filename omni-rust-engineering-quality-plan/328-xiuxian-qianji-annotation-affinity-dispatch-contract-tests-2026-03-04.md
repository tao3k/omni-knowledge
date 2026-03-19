# 328. xiuxian-qianji annotation affinity dispatch contract tests (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `tests/test_compiler_dispatch_routes.rs`
- Goal: validate not only annotation route compilation success, but also
  post-compile execution-affinity contract materialization.

## Implementation

1. Added explicit affinity contract test:
   - `compiler_dispatches_annotation_and_keeps_explicit_execution_affinity`
   - verifies `agent_id` and `role_class` parameters are preserved into
     compiled node `execution_affinity`.
2. Added derived affinity contract test:
   - `compiler_dispatches_annotation_and_derives_role_class_from_persona_id`
   - verifies `role_class` derivation from `[nodes.qianhuan].persona_id`
     (`semantic://personas/Steward.md` -> `steward`) when explicit
     role-class parameters are absent.
3. Kept integration-test boundary clean:
   - all assertions are top-level crate tests in `tests/`, with no inline
     `#[cfg(test)]` additions to production modules.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `17 passed`, `0 skipped`, `0 failed`

## Outcome

- Annotation dispatch tests now cover both route success and compiled
  affinity-data correctness.
- This reduces refactor risk around `annotation::node_execution_affinity`
  behavior by locking expected compile output contracts.
