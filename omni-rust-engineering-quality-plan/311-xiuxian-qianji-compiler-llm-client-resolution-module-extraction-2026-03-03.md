# 311. xiuxian-qianji compiler LLM client resolution module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/llm_client.rs` (new, `feature = "llm"`)
- Goal: move node-level LLM client resolution and OpenAI-compatible client
  construction out of compiler orchestration shell while preserving behavior.

## Implementation

1. Added dedicated LLM client resolution module:
   - `src/engine/compiler/llm_client.rs`
   - introduced:
     - `resolve_for_node(node_def, global_client)`
   - behavior preserved:
     - if node-level endpoint is declared, resolve endpoint via
       `llm_node::resolve_node_llm_endpoint(...)` and construct
       `OpenAIClient` with `reqwest` timeout of 300 seconds.
     - otherwise fall back to compiler-provided global LLM client.
2. Updated compiler wiring:
   - `compiler.rs` now declares `#[cfg(feature = "llm")] mod llm_client;`
   - removed `resolve_llm_client_for_node(...)` from compiler impl.
   - `dispatch_formal_audit` (`llm` build) now calls
     `llm_client::resolve_for_node(...)`.
   - `dispatch_llm` (`llm` build) now calls
     `llm_client::resolve_for_node(...)`.
3. Reduced orchestration shell density:
   - compiler core line count dropped from `206` to `183`.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/llm_client.rs`
  - result:
    - `compiler.rs`: `183`
    - `compiler/llm_client.rs`: `29`
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

- Compiler core keeps a clearer role as orchestration/dispatch layer.
- LLM endpoint/client resolution is now isolated in a domain-focused module
  for easier future evolution (for example multi-backend transport strategies)
  without re-expanding compiler core complexity.
