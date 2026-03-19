# 修仙道场 (Xiuxian Daochang) Unreadable-Literal Convergence (2026-02-27)

## Scope

Continue the `xiuxian-daochang/tests` strict-clippy convergence wave by removing
`clippy::unreadable_literal` file-level suppression markers and fixing all
surfaced long-literal warnings at source level.

## Implemented Changes

1. Removed `clippy::unreadable_literal` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced warning sites by normalizing numeric literals with
   separators, including:
   - timestamp literals in
     `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall_state.rs`
     (`1_739_900_001_888_u64`),
   - Telegram/Webhook chat-id literals in:
     - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_group_policy.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_slash_authorization.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/channels_webhook.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/channels_webhook_stress.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/channels_webhook_process.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_polling.rs`,
   - Discord attachment limits in:
     - `packages/rust/crates/xiuxian-daochang/tests/channels_discord_ingress.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/channels_discord_parsing.rs`
     (`8_388_608`).
3. Kept no-suppression-first policy:
   - no new `#[allow(...)]` was introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::unreadable_literal" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::unreadable_literal` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::unreadable_literal` with strict
pedantic and `too_many_lines` validation still green.
