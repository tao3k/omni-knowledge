# 修仙道场 (Xiuxian Daochang) Test Marker Zero Convergence (2026-02-27)

## Scope

Finalize the `xiuxian-daochang/tests` marker burndown for:

- `clippy::too_many_lines`
- `clippy::too_many_arguments`

using a no-suppression-first approach.

## Implemented Changes

1. Removed remaining marker-based file-level `allow` headers from the last 12
   test modules:
   - `packages/rust/crates/xiuxian-daochang/tests/agent/graph_executor.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_stream_consumer.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent_injection.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent_memory_gate_flow.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent_memory_persistence_backend.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_group_policy.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_webhook.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_webhook_stress.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_client.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_role_perf_smoke.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_memory.rs`
2. Added explicit crate-level test docs (`//! ...`) across touched files to
   replace `missing_docs` suppression patterns.
3. Fixed newly exposed real warning sources with structural changes instead of
   suppression:
   - split large ACL tests in
     `packages/rust/crates/xiuxian-daochang/tests/telegram_acl_overrides.rs` and
     `packages/rust/crates/xiuxian-daochang/tests/discord_acl_overrides.rs`.
   - fixed `manual_let_else` in
     `packages/rust/crates/xiuxian-daochang/tests/embedding/transport_http.rs`.
   - removed unused parameter from
     `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/bridge.rs`.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
rg -o "clippy::too_many_lines|clippy::too_many_arguments" \
  packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' \
  | sed 's/.*://g' | sort | uniq -c | sort -nr
```

Result:

- `cargo fmt -p xiuxian-daochang`: pass.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass.
- Marker scan output: empty (zero occurrences).

## Outcome

`xiuxian-daochang/tests` reached marker-zero convergence for these two categories:

- `too_many_lines`: `0`
- `too_many_arguments`: `0`

This completes the marker burndown objective while preserving the policy of
fixing root causes and docs instead of adding new suppression attributes.
