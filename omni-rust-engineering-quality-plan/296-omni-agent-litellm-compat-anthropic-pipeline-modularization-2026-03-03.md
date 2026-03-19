# 296. xiuxian-daochang litellm compat anthropic pipeline modularization (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target area: `src/llm/compat/litellm.rs`
- Goal: split mixed responsibilities and keep `litellm` runtime adapter focused
  on provider dispatch/lifecycle.

## Implementation

1. Extracted Anthropic custom-base bypass pipeline into a dedicated child module:
   - added `src/llm/compat/litellm/anthropic_custom.rs`
   - moved:
     - custom request path (`chat_anthropic_without_model_registry`)
     - request body shaping
     - message/content/tool conversion
     - custom response parsing
2. Kept runtime orchestration in parent module:
   - `LiteLlmRuntime` provider lifecycle and dispatch remain in
     `src/llm/compat/litellm.rs`
   - parent now imports `anthropic_custom::chat_anthropic_without_model_registry`
3. Import surface cleanup:
   - removed now-unneeded heavy conversion imports from parent file
   - retained shared helpers (`resolve_litellm_api_key`,
     `anthropic_messages_endpoint_from_base`) in parent for stable reuse.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-daochang/src/llm/compat/litellm.rs packages/rust/crates/xiuxian-daochang/src/llm/compat/litellm/anthropic_custom.rs`
  - result:
    - `litellm.rs`: `265`
    - `anthropic_custom.rs`: `357`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test llm --test embedding`
  - result: `38 passed`, `0 skipped`, `0 failed`

## Outcome

- `litellm` compatibility layer now follows clearer domain boundaries:
  runtime dispatch vs Anthropics custom protocol conversion.
- Behavior is preserved with targeted regression proof.
