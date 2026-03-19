# 修仙道场 (Xiuxian Daochang) Marker Burndown Wave 3: Doc-Driven Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` marker burndown for `clippy::too_many_lines` and
`clippy::too_many_arguments` using a no-suppression-first policy:

- remove stale file-level marker allows,
- replace `missing_docs` suppression with explicit crate-level test docs,
- keep strict clippy lanes green for `xiuxian-daochang`.

## Implemented Changes

1. Removed marker `allow` headers from additional small/medium test modules,
   including:
   - `packages/rust/crates/xiuxian-daochang/tests/mcp_pool_hard_timeout.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_upload.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_status.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_client_cache.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/valkey_url_precedence.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_jobs.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent_context_budget.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/llm/provider_mode.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_caption.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_injection.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall_feedback.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_discord_send.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_help.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_managed_commands.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/jobs_scheduler.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_markdown_upload.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/config_and_session.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_budget.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/gateway_http.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_markdown.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/mcp_pool_reconnect.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_webhook_process.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/gateway/http/runtime.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_discord.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_send_gate.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/reflection.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_markdown.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_resume_flow.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent_memory_scope_isolation.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_control_admin.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/transport_command_flow.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_slash_authorization.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent_context_window_recovery.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_discord_slash_authorization.rs`
2. Added explicit crate-level documentation comments (`//! ...`) in touched test
   modules to replace `missing_docs` suppression with real docs.
3. Refactored `packages/rust/crates/xiuxian-daochang/tests/agent/reflection.rs` to
   remove a real `too_many_lines` warning by extracting long-horizon fixtures
   and transition helpers, while preserving behavior and thresholds.

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
- `xiuxian-daochang` local warnings in these lanes: none.

Marker backlog in `xiuxian-daochang/tests` after this wave:

- `too_many_lines`: `42`
- `too_many_arguments`: `41`

## Outcome

This wave continued suppression-debt removal without introducing new `allow`
shortcuts, replaced missing-doc suppression with explicit docs, and kept
`xiuxian-daochang` strict clippy lanes green while reducing marker backlog by 35
entries per category from the starting snapshot (`77/76` -> `42/41`).
