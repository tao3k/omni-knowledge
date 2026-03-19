# 修仙道场 (Xiuxian Daochang) Runtime/Media/Agent Test Allow-Debt Reduction Wave (2026-02-26)

## Scope

Continue suppression-debt convergence in `xiuxian-daochang` by cleaning additional
test modules that no longer contain `expect/unwrap`, plus one hot-reload test
expectation sync to current runtime detail format.

## Why

The remaining debt was concentrated in runtime and media-support test modules
that already had explicit error handling. Removing stale suppressions there
provides immediate strict-gate signal improvement with minimal behavior risk.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` from:
   - `tests/agent/memory_recall_feedback_state.rs`
   - `tests/agent/memory_recall_metrics.rs`
   - `tests/agent/embedding_dimension.rs`
   - `tests/agent/mcp_startup.rs`
   - `tests/agent/omega/test_strategic_supervisor.rs`
   - `tests/agent/memory/decay.rs`
   - `tests/agent/memory_recall_feedback.rs`
   - `tests/agent/omega_decision.rs`
   - `tests/agent_system_prompt_injection_state.rs`
   - `tests/config_and_session.rs`
   - `tests/telegram_media_support/mod.rs`
   - `tests/telegram_media_support/bootstrap.rs`
   - `tests/telegram_media_support/media_api.rs`
   - `tests/telegram_media_support/media_api/server_bootstrap.rs`
   - `tests/telegram_media_support/media_api/routing.rs`
   - `tests/telegram_media_support/media_api/markdown_fallback.rs`
   - `tests/telegram_media_support/upload_api.rs`
   - `tests/telegram_media_support/upload_api/server_bootstrap.rs`
   - `tests/telegram_media_support/upload_api/media_group.rs`
   - `tests/telegram_media_support/upload_api/photo.rs`
2. Synced hot-reload mount assertion to current runtime detail mode string in:
   - `tests/agent/bootstrap_hot_reload.rs`
   - `mode=heyi_sync_from_disk` -> `mode=heyi_sync_incremental_or_full`

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --test agent_suite
cargo test -p xiuxian-daochang --test agent_system_prompt_injection_state
cargo test -p xiuxian-daochang --test config_and_session
cargo test -p xiuxian-daochang --test channels_telegram_media
cargo test -p xiuxian-daochang --test channels_telegram_media_caption
cargo test -p xiuxian-daochang --test channels_telegram_media_caption_fallback
cargo test -p xiuxian-daochang --test channels_telegram_media_delivery
cargo test -p xiuxian-daochang --test channels_telegram_media_markdown
cargo test -p xiuxian-daochang --test channels_telegram_media_markdown_upload
cargo test -p xiuxian-daochang --test channels_telegram_media_upload
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- All targeted test lanes passed.
- `xiuxian-daochang` remained green under pedantic clippy.
- `xiuxian-daochang/tests` allow-marker file count dropped from `76` to `56`.

## Outcome

This wave removed another large block of stale suppressions while keeping
runtime/media behavior stable and current hot-reload assertions aligned with
implementation semantics.
