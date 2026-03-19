# 315. xiuxian-qianji mechanism dispatch submodule split (`stateless` + `stateful_cfg`) (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler/mechanism_dispatch.rs`
  - `src/engine/compiler/mechanism_dispatch/stateless.rs` (new)
  - `src/engine/compiler/mechanism_dispatch/stateful_cfg.rs` (new)
- Goal: split mechanism dispatch internals by concern so task routing remains
  readable while cfg-specific branching is isolated from non-cfg paths.

## Implementation

1. Split `mechanism_dispatch` into focused submodules:
   - `mechanism_dispatch/stateless.rs`
     - `knowledge(compiler)`
     - `annotation(compiler, node_def)`
   - `mechanism_dispatch/stateful_cfg.rs`
     - `formal_audit(compiler, node_def)` with cfg-aware implementation
     - `llm(compiler, node_def)` with cfg-aware implementation
2. Simplified parent dispatch shell:
   - `mechanism_dispatch.rs` now keeps only:
     - task-type parsing (`TaskType::parse`)
     - high-level routing to `stateless` / `stateful_cfg` / `task_mechanisms`
3. Behavior parity retained:
   - preserved LLM feature-gated guard semantics.
   - preserved node-level/global LLM client fallback path.
   - preserved formal-audit and llm routing outcomes.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateless.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg.rs`
  - result:
    - `compiler.rs`: `95`
    - `mechanism_dispatch.rs`: `29`
    - `mechanism_dispatch/stateless.rs`: `18`
    - `mechanism_dispatch/stateful_cfg.rs`: `52`
- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - command attempted: `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - environment fallback used due build-directory lock contention:
    - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- Dispatch complexity is now layered:
  - `mechanism_dispatch.rs` = routing shell
  - `stateless.rs` = non-cfg branch builders
  - `stateful_cfg.rs` = cfg-sensitive routing and guards
- This reduces local cognitive load and keeps future growth from re-inflating a
  single dispatch file.
