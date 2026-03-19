# 修仙道场 (Xiuxian Daochang) Wildcard-Import and Channel-Runner Cleanup (2026-02-26)

## Scope

This shard records the next suppression-debt convergence wave in `xiuxian-daochang`,
focused on:

- eliminating all remaining `clippy::wildcard_imports`,
- eliminating `clippy::similar_names` and `clippy::too_many_arguments` in the
  Telegram node runner by restructuring the orchestration interface.

Targets:

- `packages/rust/crates/xiuxian-daochang/src/agent/feedback.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/session_context/backup.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/messages.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/context_repair.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/conversation.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall/plan.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall/execution.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall/observability.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/channel/telegram.rs`

## Changes Implemented

### 1) Removed all wildcard-import suppressions in `xiuxian-daochang`

Actions:

- Replaced all remaining `use super::*` wildcard imports with explicit imports
  for concrete symbols (for example `Agent`, `ChatMessage`, `Result`,
  `OmegaDecision`, `PolicyHintDirective`, `context_budget`, `injection`,
  `memory_recall_state`).
- Removed all `#[allow(clippy::wildcard_imports)]` attributes across
  `agent/turn_execution`, `agent/feedback`, and `agent/session_context`.
- Cleaned now-unused parent-module imports in `agent/mod.rs` that had existed
  only to feed wildcard consumers.

### 2) Telegram channel runner API reshaped to remove argument and naming debt

Actions:

- Introduced `TelegramChannelRunRequest` to replace the previous
  many-argument `run_telegram_channel_mode` signature.
- Moved ACL/slash policy composition into `run_telegram_channel_command` before
  runtime dispatch.
- Renamed local variables to reduce near-collision naming and improve clarity
  (for example bind/path/backend/token/secret resolution variables).
- Removed:
  - `#[allow(clippy::similar_names)]` on `run_telegram_channel_command`
  - `#[allow(clippy::similar_names, clippy::too_many_arguments)]` on
    `run_telegram_channel_mode`

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
cargo test -p xiuxian-daochang --lib
rg -n "allow\\(clippy::wildcard_imports\\)" \
  packages/rust/crates/xiuxian-daochang/src \
  packages/rust/crates/xiuxian-daochang/tests
rg -o "allow\\(clippy::[a-z0-9_]+(?:, clippy::[a-z0-9_]+)*\\)" \
  packages/rust/crates/xiuxian-daochang/src \
  packages/rust/crates/xiuxian-daochang/tests \
| sed -E 's/.*allow\\(//; s/\\)//' | tr ',' '\\n' \
| sed -E 's/^\\s*clippy:://; s/^\\s+|\\s+$//g' | sort | uniq -c | sort -nr
```

Results:

- `cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic`: pass for
  `xiuxian-daochang` (workspace still reports existing warnings in other crates,
  outside this shard scope).
- `cargo test -p xiuxian-daochang --lib`: pass (`224 passed`, `0 failed`, `8 ignored`).
- `wildcard_imports` suppression category: eliminated (`0` remaining).
- `similar_names` and `too_many_arguments` suppression categories:
  eliminated (`0` remaining).
- Updated suppression inventory in `xiuxian-daochang`:
  - `struct_field_names`: 3
  - `cast_precision_loss`: 3
  - `cast_sign_loss`: 1
  - `cast_possible_truncation`: 1

## Outcome

- `xiuxian-daochang` suppression debt is now concentrated only in
  domain-model naming and cast-precision categories.
- Import boundaries are explicit and locally traceable.
- Telegram channel runtime orchestration API is cleaner and more maintainable
  while preserving runtime behavior.
