# 修仙道场 (Xiuxian Daochang) Type-Complexity and Unnecessary-Wraps Convergence (2026-02-26)

## Scope

This shard records the next `xiuxian-daochang` suppression-debt reduction wave focused
on:

- removing remaining `clippy::type_complexity` suppressions,
- removing `clippy::unnecessary_wraps` suppressions without re-adding blanket
  allows,
- keeping runtime initialization semantics explicit and test-verified.

Targets:

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/run_polling/channel_listener.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/acl_config.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/constructor/entrypoints.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/webhook/builders/core.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/channel/constructors.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/ingress.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels_control_command_authorization.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels_discord.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels_discord_slash_authorization.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_group_policy.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_slash_authorization.rs`
- `packages/rust/crates/xiuxian-daochang/tests/discord_acl_overrides.rs`
- `packages/rust/crates/xiuxian-daochang/tests/telegram_acl_overrides.rs`

## Changes Implemented

### 1) Removed remaining `type_complexity` suppressions

Actions:

- Introduced `PollingListenerRuntime` alias in Telegram polling listener startup
  return type.
- Introduced `TelegramSlashAclOverrides` alias for ACL slash override tuple
  return type.
- Deleted both `#[allow(clippy::type_complexity)]` attributes.

### 2) Removed constructor-level `unnecessary_wraps` suppressions

Actions:

- Converted these constructors from `anyhow::Result<Self>` to `Self`:
  - `TelegramChannel::new_with_partition_and_control_command_policy`
  - `TelegramChannel::new_with_partition_and_admin_users_and_control_command_allow_from_and_command_rules`
  - `DiscordChannel::new_with_control_command_policy`
  - `DiscordChannel::new_with_partition_and_control_command_policy`
- Updated source and test call sites to remove stale `?`/`.expect(...)` chains.
- Added `#[must_use]` on the newly infallible constructor surfaces.

### 3) Preserved fallible runtime boundaries with explicit validation

Actions:

- Kept startup/ingress builders returning `Result` where runtime initialization
  is expected to fail fast on invalid config.
- Added explicit non-empty bot-token validation:
  - Telegram polling listener startup: `ensure!(!bot_token.trim().is_empty(), ...)`
  - Discord ingress app builder: `ensure!(!bot_token.trim().is_empty(), ...)`

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
cargo test -p xiuxian-daochang --lib
cargo test -p xiuxian-daochang --test channels_discord_ingress
cargo test -p xiuxian-daochang --test channels_telegram_polling
cargo test -p xiuxian-daochang --test channels_discord --test channels_discord_slash_authorization --test channels_control_command_authorization --test channels_telegram_slash_authorization --test telegram_acl_overrides
rg -o "allow\\(clippy::[a-z0-9_]+(?:, clippy::[a-z0-9_]+)*\\)" \
  packages/rust/crates/xiuxian-daochang/src \
  packages/rust/crates/xiuxian-daochang/tests \
| sed -E 's/.*allow\\(//; s/\\)//' | tr ',' '\\n' \
| sed -E 's/^\\s*clippy:://; s/^\\s+|\\s+$//g' | sort | uniq -c | sort -nr
```

Results:

- `cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic`: pass for
  `xiuxian-daochang` (workspace still shows existing `xiuxian-zhixing` warning
  outside this scope).
- `cargo test -p xiuxian-daochang --lib`: pass (`224 passed`, `0 failed`, `8 ignored`).
- Targeted integration tests: pass (`channels_discord_ingress`,
  `channels_telegram_polling`, slash/control/ACL suites).
- `xiuxian-daochang` suppression inventory now:
  - `wildcard_imports`: 11
  - `large_types_passed_by_value`: 4
  - `struct_field_names`: 3
  - `cast_precision_loss`: 3
  - `similar_names`: 2
  - `too_many_arguments`: 1
  - `cast_sign_loss`: 1
  - `cast_possible_truncation`: 1

## Outcome

- `type_complexity` suppressions in `xiuxian-daochang` are eliminated.
- `unnecessary_wraps` suppressions in `xiuxian-daochang` are eliminated.
- Constructor and runtime startup boundaries are clearer:
  infallible constructors remain infallible; runtime bootstrap paths fail fast
  on invalid configuration.
