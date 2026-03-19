# 281. xiuxian-daochang managed-runtime/redis/metrics test remap elimination wave (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Goal: remove remaining `include!("../src/...")` source remaps from three
  top-level integration test lanes using stable `xiuxian_daochang::test_support`
  adapters.

## Why

- Source remap tests bypass crate boundaries and couple tests to internal file
  layout.
- We standardize on package-top `tests/` with stable public/test-support
  contracts.
- This aligns with modern Rust engineering hygiene: explicit boundaries,
  minimal shim logic, and behavior-preserving migration.

## Implementation

1. Added new stable test-support adapters:
   - `src/test_support/managed_runtime.rs`
   - `src/test_support/session_redis.rs`
   - `src/test_support/memory_metrics.rs`
2. Exported adapters in `src/test_support/mod.rs`.
3. Opened crate-internal visibility only where required for adapter access:
   - `src/channels/mod.rs`: `managed_runtime` -> `pub(crate) mod managed_runtime`
   - `src/session/mod.rs`: `redis_backend` -> `pub(crate) mod redis_backend`
   - `src/session/redis_backend/mod.rs`: `message_store` -> `pub(crate) mod message_store`
   - `src/agent/mod.rs`: `memory_recall_metrics` -> `pub(crate) mod memory_recall_metrics`
4. Added crate-internal bridge helpers:
   - `src/session/redis_backend/message_store.rs`:
     `test_encode_chat_message_payload`, `test_decode_chat_message_payload`
   - `src/agent/memory_recall_metrics.rs`:
     `observe_*`, `snapshot`, `ratio_as_f32` promoted to `pub(crate)` to avoid
     source remap and keep logic single-sourced.
5. Replaced source-remap harnesses with package-top module entrypoints:
   - `tests/channels_managed_runtime_unit.rs`
   - `tests/session_redis_backend_unit.rs`
   - `tests/agent_memory_recall_metrics.rs`
6. Migrated test imports to `xiuxian_daochang::test_support`:
   - `tests/channels/managed_runtime/tests/test_session_partition_persistence.rs`
   - `tests/channels/managed_runtime/tests/test_turn.rs`
   - `tests/session/redis_backend/tests.rs`
   - `tests/agent/memory_recall_metrics_impl/tests.rs`

## Verification

- Remap debt scan:
  - `rg -n "include!\\(\"\\.\\./src/|#\\[path\\s*=\\s*\"\\.\\./src/|#\\[path\\s*=\\s*\"\\.\\./\\.\\./src/" packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l`
  - result: `31 -> 27` (4 remaps removed in this wave)
- Targeted tests:
  - `cargo nextest run -p xiuxian-daochang --test channels_managed_runtime_unit --test session_redis_backend_unit --test agent_memory_recall_metrics`
  - result: `15 passed, 0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass

## Remaining remap debt after this wave

- `embedding.rs`
- `agent_memory_stream_consumer_unit.rs`
- `agent_memory_recall_state_unit.rs`
- `channels_telegram_runtime_unit.rs`
- `channels_discord_runtime_unit.rs`
- `nodes_warmup.rs`
- `agent_bootstrap.rs`
- `agent_zhenfa_unit.rs`
- `agent_session_context_unit.rs`

## Next slice recommendation

- Continue with smallest/high-confidence units first:
  1. `agent_memory_recall_state_unit`
  2. `agent_memory_stream_consumer_unit`
  3. `nodes_warmup`
- Then handle large channel runtime suites (`discord`/`telegram`) with
  incremental adapter extraction to avoid destabilizing broad test matrices.
