# 202. 修仙道场 (Xiuxian Daochang) Src Allow-Cleanup Root-Cause Wave (2026-02-28)

## Scope

This wave continued the suppression-debt policy for `xiuxian-daochang` production
sources, focusing on `#[allow(dead_code)]` and `#[allow(unused_imports)]`
removal by fixing call paths, visibility boundaries, and compatibility parsing
behavior.

## What Changed

1. Removed no-longer-needed dead-code suppressions in core channel contracts:
   - `packages/rust/crates/xiuxian-daochang/src/channels/traits.rs`

2. Converted Telegram runtime test-only preview export to explicit test scope:
   - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/api.rs`
   - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/mod.rs`
   - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/observability/mod.rs`

3. Removed response-shape dead field suppression by dropping the unused
   `finish_reason` field from HTTP completion decoding:
   - `packages/rust/crates/xiuxian-daochang/src/llm/types.rs`

4. Reflection lane: removed dead-code suppression by making render helpers
   actively consumed in runtime hint logging:
   - `packages/rust/crates/xiuxian-daochang/src/agent/reflection/turn.rs`
   - `packages/rust/crates/xiuxian-daochang/src/agent/reflection/mod.rs`
   - `packages/rust/crates/xiuxian-daochang/src/agent/reflection_runtime_state.rs`
   - `packages/rust/crates/xiuxian-daochang/src/agent/reflection/lifecycle.rs`

5. Agent builder/state cleanup:
   - Removed dead-code suppressions on active fields.
   - Renamed hot-reload retention field to `_hot_reload_driver` (explicit
     ownership/lifetime intent without suppression).
   - Files:
     - `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`
     - `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/builder.rs`

6. Legacy session backup compatibility parser now consumes decoded metadata
   fields and emits observability instead of carrying dead fields:
   - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/message_store.rs`

7. Observability event registry warning convergence:
   - Kept canonical `SessionEvent::ALL` in production source.
   - Added symbol probe (zero runtime behavior impact) to ensure variants and
     registry constant stay compiler-visible without suppressions.
   - File: `packages/rust/crates/xiuxian-daochang/src/observability/session_events.rs`

## Validation Evidence

Commands executed:

1. `cargo fmt -p xiuxian-daochang`
2. `CARGO_TARGET_DIR=target/clippy-xiuxian-daochang cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic -W clippy::too_many_lines`
3. `CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang`

Outcomes:

- Strict clippy for `xiuxian-daochang` passed with no warnings/errors under the
  specified flags.
- `nextest` summary: `653 passed`, `0 failed`, `30 skipped`.

## Remaining Allow Inventory (src)

After this wave, `xiuxian-daochang/src` allow-usage is limited to:

1. Narrow `#[allow(unsafe_code)]` in test-only modules under
   `src/agent/bootstrap/tests.rs` and `src/config/tests.rs`.
2. Targeted numeric-cast allowance in
   `src/agent/embedding_dimension.rs`.

No broad file/module-level dead-code or unused-import suppression remains in
production runtime sources touched by this wave.
