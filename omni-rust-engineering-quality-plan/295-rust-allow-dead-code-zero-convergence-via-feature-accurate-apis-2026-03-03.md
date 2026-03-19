# 295. rust allow(dead_code) zero convergence via feature-accurate APIs (2026-03-03)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji`
- Goal: remove remaining `allow(dead_code)` suppressions and replace them with
  feature-accurate structure.

## Implementation

1. `xiuxian-llm` provider modules:
   - `src/llm/providers/openai.rs`
   - `src/llm/providers/anthropic.rs`
   - `src/llm/providers/minimax.rs`
   - Removed non-feature fallback aliases/functions that existed only to avoid
     dead-code warnings.
   - Kept provider type aliases and builders strictly under
     `feature = "provider-litellm"`.
2. `xiuxian-qianji` compiler:
   - `src/engine/compiler.rs`
   - Removed field-level `allow(dead_code)` on `llm_client`.
   - Split `QianjiCompiler::new(...)` into feature-specific constructors:
     - `#[cfg(feature = "llm")]` stores `llm_client`
     - `#[cfg(not(feature = "llm"))]` ignores `_llm_client` parameter without
       suppression

## Verification

- Suppression audit:
  - `rg -n "allow\\(dead_code\\)" packages/rust/crates -g "*.rs"`
  - result: no matches (`0`)
- Mandatory touched-crate clippy gates:
  - `cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass for both crates
- Targeted regression:
  - `cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_smoke`
  - result: `9 passed`, `1 skipped`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- `allow(dead_code)` suppressions are eliminated across Rust crates.
- Feature boundaries now encode intent directly in structure and signatures.
