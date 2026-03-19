# 299. xiuxian-qianji compiler llm-node module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area: `src/engine/compiler.rs`
- Goal: extract node-level `llm` endpoint/config parsing and analyzer input
  assembly out of the compiler core flow.

## Implementation

1. Added dedicated LLM-node helper module:
   - `src/engine/compiler/llm_node.rs`
   - includes:
     - node-level endpoint/provider resolution
     - API-key env fallback resolution
     - `llm` analyzer mechanism config assembly
2. Updated compiler wiring:
   - `compiler.rs` declares `#[cfg(feature = "llm")] mod llm_node;`
   - `resolve_llm_client_for_node` now calls
     `llm_node::resolve_node_llm_endpoint(...)`
   - `build_llm_mechanism` now uses `llm_node::mechanism_config(...)`
3. Removed duplicated in-file helper cluster:
   - deleted `llm_model`, `llm_provider_kind`, dedicated-endpoint checks,
     and node-level endpoint resolver from `compiler.rs`.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/formal_audit.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/llm_node.rs`
  - result:
    - `compiler.rs`: `778`
    - `compiler/formal_audit.rs`: `94`
    - `compiler/llm_node.rs`: `145`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`
- Global suppression audit:
  - `rg -n "#\\[allow\\(|#!\\[allow\\(|cfg_attr\\([^\\)]*allow\\(" packages/rust/crates -g "*.rs" | wc -l`
  - result: `0`

## Outcome

- Compiler has a clearer orchestration boundary.
- LLM node configuration logic is now isolated and independently maintainable.
