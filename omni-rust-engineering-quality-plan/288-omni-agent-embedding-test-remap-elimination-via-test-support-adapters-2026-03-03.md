# 288. xiuxian-daochang embedding test remap elimination via test-support adapters (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target lane: `tests/embedding.rs`
- Goal: remove four source includes (`backend`, `transport_litellm`,
  `transport_openai`, `transport_http`) and converge to package-top
  integration tests with stable `test_support` contracts.

## Implementation

1. Added embedding test-support adapter module:
   - `src/test_support/embedding.rs`
   - Exposed:
     - `parse_embedding_client_backend_mode`
     - `EmbeddingBackendMode`
     - `embed_http`
     - litellm-only normalization helpers and placeholder key constant
2. Wired test-support exports:
   - `src/test_support/mod.rs`
3. Added crate-internal embedding test adapters:
   - `src/embedding/mod.rs`
   - `test_parse_backend_mode`
   - `test_embed_http`
   - litellm test adapters (`TEST_OLLAMA_PLACEHOLDER_API_KEY`, normalization wrappers)
4. Added backend test bridge:
   - `src/embedding/backend.rs`
   - `test_parse_backend_mode`
5. Added litellm test bridges:
   - `src/embedding/transport_litellm.rs`
   - `TEST_OLLAMA_PLACEHOLDER_API_KEY`
   - normalization test helper wrappers
6. Replaced source-include harness with standard package-top entrypoint:
   - `tests/embedding.rs`
   - now imports `xiuxian_daochang::test_support` and mounts
     `tests/embedding/*.rs` via `#[path = ...]`.
7. Migrated backend lane imports to stable adapter naming:
   - `tests/embedding/backend.rs`

## Verification

- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test embedding`
  - result: `8 passed`, `0 skipped`, `0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass
- Remap debt counter:
  - `rg -n "include!\(\"\.\./src/|#\[path\s*=\s*\"\.\./src/|#\[path\s*=\s*\"\.\./\.\./src/" packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l`
  - result: `19 -> 15`
