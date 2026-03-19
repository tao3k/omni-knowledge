# 修仙道场 (Xiuxian Daochang) LiteLLM Compatibility Layer (2026-02-24)

## Objective

Implement an internal compatibility adapter for `litellm-rs` inside
`xiuxian-daochang` so upstream dependency constraints (`reqwest 0.11` chain) do not
block local engineering quality improvements.

The target was to preserve runtime behavior while improving isolation and
future upgrade readiness.

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Focus: `llm` integration boundary, not provider behavior changes.
- Non-goal: forcing immediate `reqwest` single-version convergence across all
  transitive dependencies.

## Implemented Changes

1. Added a dedicated compatibility namespace:
   - `packages/rust/crates/xiuxian-daochang/src/llm/compat/mod.rs`
   - `packages/rust/crates/xiuxian-daochang/src/llm/compat/litellm.rs`

2. Extracted `litellm-rs` lifecycle and dispatch from `llm/client.rs`:
   - provider initialization caching (`OnceCell`) moved into
     `LiteLlmRuntime`,
   - request construction and response conversion moved into adapter module,
   - API key fallback resolution moved into adapter module.

3. Kept `LlmClient` as orchestrator:
   - HTTP path remains in `client.rs`,
   - LiteLLM path now dispatches through `LiteLlmDispatchConfig` +
     `LiteLlmRuntime`,
   - feature-gated compile behavior remains unchanged.

4. Updated module wiring:
   - `packages/rust/crates/xiuxian-daochang/src/llm/mod.rs` now declares
     `mod compat;`.

## Why This Matters

- Reduces direct surface area coupled to upstream `litellm-rs` internals.
- Makes future backend and dependency migration work more incremental:
  only the adapter must change first, not all call sites.
- Preserves current default behavior while improving maintainability for
  profile-split builds.

## Validation Evidence

All commands were executed from repository root on 2026-02-24.

1. Formatting:
   - `cargo fmt -p xiuxian-daochang`
   - Result: pass.

2. Default profile compile:
   - `cargo check -p xiuxian-daochang`
   - Result: pass.

3. Reduced profile compile:
   - `cargo check -p xiuxian-daochang --no-default-features`
   - Result: pass.

4. Default profile test build:
   - `cargo test -p xiuxian-daochang --no-run`
   - Result: pass.

5. Reduced profile test build:
   - `CARGO_TARGET_DIR=/tmp/xiuxian-daochang-no-default cargo test -p xiuxian-daochang --no-run --no-default-features`
   - Result: pass.
   - Note: dedicated `CARGO_TARGET_DIR` was used to avoid local artifact-lock
     contention from concurrent background builds.

## Next Slice

1. Keep transitive `reqwest 0.11` as monitored advisory until upstream moves.
2. Continue narrowing `litellm-rs` usage to adapter-only files.
3. Prepare a follow-up migration slice for workspace-side `reqwest 0.13.x`
   trial after adapter boundaries stabilize.
