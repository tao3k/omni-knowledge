# 330. xiuxian-qianji llm-feature dispatch route tests and cfg-guard fix (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `tests/test_compiler_dispatch_routes_llm.rs` (new)
  - `src/engine/compiler/stateful_mechanisms.rs`
- Goal: add explicit `--features llm` dispatch route coverage and remove a
  feature-specific dead-code warning by tightening cfg boundaries.

## Implementation

1. Added dedicated `llm`-feature route tests:
   - new file: `tests/test_compiler_dispatch_routes_llm.rs`
   - coverage:
     - `llm` task compiles with a global stub LLM client.
     - `formal_audit` with node-level llm binding compiles in augmented mode
       with a global stub client.
     - `llm` task fails with clear error when global client is missing.
2. Fixed feature-specific dead code in compiler internals:
   - added `#[cfg(not(feature = "llm"))]` to
     `formal_audit_requires_llm_guard` in
     `src/engine/compiler/stateful_mechanisms.rs`.
   - rationale: function is only referenced in non-`llm` dispatch path and
     should not be compiled in `llm` builds.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate (default feature set):
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Additional feature-lane lint verification:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji --features llm -- -W clippy::too_many_lines`
  - result: pass for `xiuxian-qianji` with dead-code warning removed; upstream
    dependency crate `xiuxian-llm` emits unrelated warnings.
- Targeted regression (default feature set):
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 skipped`, `0 failed`
- Targeted regression (`llm` feature set):
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 skipped`, `0 failed`

## Outcome

- Compiler dispatch contract coverage now spans both default and `llm` feature
  lanes with explicit integration tests.
- `xiuxian-qianji` no longer reports the feature-specific dead-code warning in
  `llm` builds for this path.
