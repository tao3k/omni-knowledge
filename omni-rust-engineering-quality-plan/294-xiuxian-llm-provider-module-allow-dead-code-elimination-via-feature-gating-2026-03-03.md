# 294. xiuxian-llm provider-module allow(dead_code) elimination via feature gating (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-llm`
- Target area: `src/llm/providers/{openai,anthropic,minimax}.rs`
- Goal: remove local `allow(dead_code)` suppressions and converge to explicit
  feature-gated compilation boundaries.

## Implementation

1. Removed non-feature fallback aliases and builder stubs:
   - deleted `#[cfg(not(feature = "provider-litellm"))]` aliases to `()`
   - deleted non-feature async builder stubs returning `ProviderUnavailable`
2. Kept only feature-enabled provider symbols:
   - `LiteLlmOpenAIProvider` + `build_openai_provider(...)`
   - `LiteLlmAnthropicProvider` + `build_anthropic_provider(...)`
   - `LiteLlmMinimaxProvider` + `build_minimax_provider(...)`
3. Tightened imports with `cfg`:
   - `LlmError`/`LlmResult` imports now compile only under
     `feature = "provider-litellm"`.
4. Removed all local suppression attributes:
   - no remaining `allow(dead_code)` in `xiuxian-llm/src`.

## Verification

- Suppression audit:
  - `rg -n "allow\\(dead_code\\)" packages/rust/crates/xiuxian-llm/src -g "*.rs"`
  - result: no matches (`0`)
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_smoke`
  - result: `9 passed`, `1 skipped`, `0 failed`

## Outcome

- Provider modules now rely on feature-accurate compilation instead of
  dead-code suppression.
- The crate keeps behavior intact while reducing lint debt and improving
  readability of feature boundaries.
