# 283. xiuxian-daochang memory-stream-consumer remap elimination via test-support boundary (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target lane: `tests/agent_memory_stream_consumer_unit.rs`
- Goal: remove source includes:
  - `../src/agent/logging/repeated_failure.rs`
  - `../src/agent/memory_stream_consumer/mod.rs`

## Implementation

1. Added crate-internal test adapters in production module boundary:
   - `src/agent/memory_stream_consumer/mod.rs`
   - Introduced test-facing bridge types:
     - `TestMemoryStreamEvent`
     - `TestMemoryStreamConsumerRuntimeConfig`
     - `TestStreamReadErrorKind`
   - Added wrapper functions for parse/read/ack/promotion/error-classification and
     stream timing/config helpers.
2. Opened minimal visibility for adapter consumption:
   - `src/agent/mod.rs`: `memory_stream_consumer` -> `pub(crate) mod ...`
3. Added stable public test-support adapter:
   - `src/test_support/memory_stream_consumer.rs`
   - Exposes integration-safe API:
     - `parse_xreadgroup_reply`, `read_stream_events`, `ensure_consumer_group`
     - `ack_and_record_metrics`, `queue_promoted_candidate`
     - `classify_stream_read_error`, `StreamReadErrorKind`
     - `should_surface_repeated_failure`
     - `build_consumer_name`, `compute_retry_backoff_ms`
     - timeout/config/redis-error helpers.
4. Exported adapter from `src/test_support/mod.rs`.
5. Replaced top-level include harness:
   - `tests/agent_memory_stream_consumer_unit.rs` -> package-top module entrypoint.
6. Rewired test imports:
   - `tests/agent/memory_stream_consumer/tests.rs` now consumes
     `xiuxian_daochang::test_support` only.

## Verification

- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test agent_memory_stream_consumer_unit --test nodes_warmup --test channels_managed_runtime_unit --test session_redis_backend_unit --test agent_memory_recall_metrics`
  - result: `32 passed`, `4 skipped`, `0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass
  - note: unrelated upstream warning observed in dependency `xiuxian-wendao/src/ingress/spider.rs` (`cast_possible_truncation`), outside this touched scope.
- Remap debt counter:
  - `rg -n "include!\\(\"\\.\\./src/|#\\[path\\s*=\\s*\"\\.\\./src/|#\\[path\\s*=\\s*\"\\.\\./\\.\\./src/" packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l`
  - result: `26 -> 24`

## Remaining remap lanes

- `agent_memory_recall_state_unit.rs`
- `agent_session_context_unit.rs`
- `agent_zhenfa_unit.rs`
- `agent_bootstrap.rs`
- `embedding.rs`
- `channels_discord_runtime_unit.rs`
- `channels_telegram_runtime_unit.rs`
