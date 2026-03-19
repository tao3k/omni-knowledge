# 362. xiuxian-qianji executors formal-audit directory moduleization and native/llm path split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/executors/formal_audit.rs`
  - added:
    - `src/executors/formal_audit/mod.rs`
    - `src/executors/formal_audit/native.rs`
    - `src/executors/formal_audit/llm.rs`
- Goal:
  - replace mixed `formal_audit` implementation with concern-split directory
    modules while preserving public mechanism APIs and behavior across native
    and llm-feature execution paths.

## Implementation

1. Converted `executors::formal_audit` to directory module:
   - `mod.rs` is interface-only and re-exports:
     - `FormalAuditMechanism`
     - `LlmAugmentedAuditMechanism` (feature-gated)
2. Split responsibilities by concern:
   - `native.rs`:
     - `FormalAuditMechanism` invariant-check path and retry/continue decisions.
   - `llm.rs`:
     - `LlmAugmentedAuditMechanism` critique generation, score parsing,
       threshold/retry-budget control, memrl signal shaping.
     - llm-only helper functions (`context_non_empty_string`,
       `resolve_model_for_request`, xml score extraction and reward mapping).
3. API compatibility preserved:
   - external import paths remain:
     - `xiuxian_qianji::executors::formal_audit::FormalAuditMechanism`
     - `xiuxian_qianji::executors::formal_audit::LlmAugmentedAuditMechanism`
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/executors/formal_audit/mod.rs packages/rust/crates/xiuxian-qianji/src/executors/formal_audit/native.rs packages/rust/crates/xiuxian-qianji/src/executors/formal_audit/llm.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Formal-audit focused regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test executors_formal_audit --test test_formal_adversarial_audit`
  - result: `3 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test llm_augmented_formal_audit --test test_compiler_dispatch_routes_llm`
  - result: `7 passed`, `0 failed`
- Core qianji regression lane:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`

## Outcome

- `formal_audit` now follows interface-only entrypoint and clear native/llm
  execution boundaries.
- Formal-audit mechanism behavior remains stable across native and llm
  execution profiles.
