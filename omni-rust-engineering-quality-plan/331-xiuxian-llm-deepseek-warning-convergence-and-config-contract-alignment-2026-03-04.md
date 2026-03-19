# 331. xiuxian-llm deepseek warning convergence and config contract alignment (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (revalidation only)
- Target area:
  - `src/llm/vision/deepseek/inference.rs`
  - `src/llm/vision/deepseek/native/engine.rs`
  - `src/test_support.rs`
  - `tests/llm_vision_deepseek_config_unit.rs`
- Goal: eliminate new clippy warnings introduced during deepseek OCR refactor
  without suppression, and align test expectations with current embedded config
  defaults.

## Implementation

1. Fixed `doc_markdown` warnings:
   - updated docs to use backticks for `DeepSeek` in:
     - `src/llm/vision/deepseek/inference.rs`
     - `src/test_support.rs`
2. Fixed `field_reassign_with_default`:
   - replaced post-init assignment on `DecodeParameters` with struct-literal
     initialization using `..Default::default()`.
3. Refactored deepseek OCR inference for line-budget and readability:
   - split `infer_markdown` into smaller focused units:
     - cache probe path
     - uncached decode path
     - cache store helper
     - inference telemetry/log helper
   - removed `too_many_lines` hotspot without `allow(...)`.
4. Fixed follow-up pedantic warnings from the refactor:
   - removed unused `self` receiver on cache-probe helper.
   - replaced high-arity logging helper args with `InferenceTelemetry` struct.
   - collapsed nested `if` in image payload decode.
5. Preserved decode payload precedence:
   - explicit order remains `original -> resized -> grayscale`.
6. Aligned config test contract with embedded default file:
   - `tests/llm_vision_deepseek_config_unit.rs` updated
     `max_new_tokens` expectation from `512` to `64`, matching
     `resources/config/vision_deepseek.toml`.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gates:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji --features llm -- -W clippy::too_many_lines`
  - result: pass (no remaining warnings from this change set)
- Targeted `xiuxian-llm` deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit`
  - result: `15 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `xiuxian-llm` deepseek lane is back to warning-clean under strict clippy
  gates without suppression.
- Deepseek OCR inference code is more modular and easier to maintain.
- Deepseek config test contract now matches the repository's canonical embedded
  default (`max_new_tokens = 64`).
