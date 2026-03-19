# 修仙道场 (Xiuxian Daochang) Feature-Gated Profile Validation (2026-02-24)

## Objective

Convert the recent dependency-gating edits for `xiuxian-daochang` into a real,
reproducible profile split with compile/test evidence:

- default profile (current production behavior),
- reduced profile (`--no-default-features`) with `litellm-rs` disabled.

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Focus area: compile-time capability isolation for LLM integration path
  (`litellm-rs`) without breaking current runtime defaults.

## Implemented Changes

1. Added real `cfg(feature = "agent-provider-litellm")` boundaries:
   - `src/llm/mod.rs`
   - `src/llm/client.rs`
   - `src/llm/tools.rs`
   - `src/llm/backend.rs`
   - `src/llm/providers/mod.rs`
   - `src/llm/providers/openai/mod.rs`
   - `src/llm/providers/minimax/mod.rs`
   - `src/embedding/mod.rs`
   - `src/embedding/backend.rs`
   - `src/embedding/client.rs`

2. Added feature-off fallback behavior:
   - LLM backend parser falls back to HTTP when `litellm-rs` is disabled.
   - Embedding backend parser falls back to HTTP when `litellm-rs` is disabled.
   - Runtime dispatch in embedding path degrades to `http -> mcp` fallback
     instead of failing unresolved imports.

3. Cleaned feature-off warnings to keep reduced profile maintainable:
   - conditional imports/re-exports,
   - conditional fields/functions,
   - targeted dead-code allowances for feature-off-only symbols.

4. Dependency decision for this wave:
   - `serenity` was kept as a normal dependency (not gated yet) to avoid
     mixing Discord runtime stubbing work into this LLM-lane slice.
   - `litellm-rs` remains optional and is now truly disable-able.

## Validation Evidence

All commands run from repository root on 2026-02-24.

1. Reduced profile compile:
   - `CARGO_TARGET_DIR=target/codex-agent-nodflt-check cargo check -p xiuxian-daochang --no-default-features`
   - Result: pass.

2. Reduced profile test build:
   - `CARGO_TARGET_DIR=target/codex-agent-nodflt-check cargo test -p xiuxian-daochang --no-run --no-default-features`
   - Result: pass.

3. Default profile compile:
   - `CARGO_TARGET_DIR=target/codex-agent-default-check cargo check -p xiuxian-daochang`
   - Result: pass.

4. Default profile test build:
   - `CARGO_TARGET_DIR=target/codex-agent-default-check cargo test -p xiuxian-daochang --no-run`
   - Result: pass.

## Impact On Modernization Plan

This closes a key prerequisite from
`19-reqwest011-transitive-decommission-plan-2026-02-24.md`:

- capability split is now enforceable at compile time for the `litellm-rs` path.

It does **not** yet remove `reqwest 0.11` from the default graph, because default
profile still enables the `litellm-rs` integration.

## Next Slice

1. CI matrix integration for profile split is completed in:
   - `22-xiuxian-daochang-profile-matrix-ci-integration-2026-02-24.md`.
2. Continue abstraction work from plan `19`:
   - route core LLM operations through internal HTTP boundary (`reqwest 0.12` path),
   - keep `litellm-rs` as optional compatibility adapter.
3. Re-open Discord dependency isolation as a separate slice after LLM boundary
   progress is stable.
