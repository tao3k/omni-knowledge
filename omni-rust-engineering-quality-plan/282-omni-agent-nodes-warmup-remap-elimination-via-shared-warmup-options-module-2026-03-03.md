# 282. xiuxian-daochang nodes warmup remap elimination via shared warmup-options module (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target lane: `tests/nodes_warmup.rs`
- Goal: remove `include!("../src/nodes/warmup.rs")` remap while preserving
  exact warmup option precedence behavior.

## Core refactor

1. Added shared library module:
   - `src/warmup_options.rs`
   - Contains canonical warmup resolution data model and logic:
     - `WarmupEnvOverrides`
     - `WarmupOptions`
     - `resolve_warmup_options(...)`
2. Exposed module from library boundary:
   - `src/lib.rs`: `#[doc(hidden)] pub mod warmup_options;`
3. Rewired binary node warmup to consume shared logic:
   - `src/nodes/warmup.rs`
   - Kept runtime env collection in binary (`warmup_env_overrides_from_process_env`)
   - Delegated option resolution to `xiuxian_daochang::warmup_options::resolve_warmup_options`
4. Added stable test-support passthrough:
   - `src/test_support/warmup.rs`
   - Re-exports shared warmup options API for integration tests.
5. Removed include-based test harness:
   - Replaced `tests/nodes_warmup.rs` with package-top module entrypoint.
   - Updated `tests/nodes/warmup_impl/tests.rs` imports to
     `xiuxian_daochang::test_support::{...}`.

## Why this structure is better

- Single-source option precedence logic is now shared by:
  - runtime CLI node (`src/nodes/warmup.rs`)
  - integration tests (`tests/nodes/warmup_impl/tests.rs`)
- Eliminates drift risk between runtime code and test-only copied logic.
- Keeps binary-only env probing concerns separate from reusable resolution logic.

## Verification

- Targeted tests:
  - `cargo nextest run -p xiuxian-daochang --test nodes_warmup --test channels_managed_runtime_unit --test session_redis_backend_unit --test agent_memory_recall_metrics`
  - result: `18 passed, 0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass
- Remap debt counter:
  - `rg -n "include!\\(\"\\.\\./src/|#\\[path\\s*=\\s*\"\\.\\./src/|#\\[path\\s*=\\s*\"\\.\\./\\.\\./src/" packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l`
  - result: `27 -> 26` for this wave (cumulative from previous wave still retained).

## Remaining high-priority remap lanes

- `agent_memory_stream_consumer_unit.rs`
- `agent_memory_recall_state_unit.rs`
- `agent_session_context_unit.rs`
- `agent_zhenfa_unit.rs`
- `agent_bootstrap.rs`
- `embedding.rs`
- `channels_discord_runtime_unit.rs`
- `channels_telegram_runtime_unit.rs`
