# 308. xiuxian-qianji compiler cfg dispatch shell convergence (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area: `src/engine/compiler.rs`
- Goal: remove inline `cfg(feature = "llm")` branching noise from
  `build_mechanism` by introducing thin dispatch shells for feature-gated lanes.

## Implementation

1. Added thin dispatch shells:
   - `dispatch_formal_audit(&self, node_def)`
   - `dispatch_llm(&self, node_def)`
2. Kept feature gating at method boundary:
   - `#[cfg(feature = "llm")]` variants delegate to feature-enabled builders.
   - `#[cfg(not(feature = "llm"))]` variants preserve fallback behavior with
     identical error contract for `llm` task type.
3. Simplified typed task dispatch:
   - `build_mechanism` now routes `FormalAudit` and `Llm` to dispatch shells
     instead of carrying inline cfg blocks inside the central match.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs`
  - result:
    - `compiler.rs`: `384`
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

- `build_mechanism` is now a cleaner typed dispatcher with less conditional noise.
- Feature-gated behavior remains stable while dispatch readability and maintenance
  ergonomics improve.
