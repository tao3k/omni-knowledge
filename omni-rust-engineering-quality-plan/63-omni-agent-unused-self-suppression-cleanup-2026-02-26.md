# 修仙道场 (Xiuxian Daochang) Unused-Self Suppression Cleanup (2026-02-26)

## Scope

This shard records a focused cleanup wave that removed `clippy::unused_self`
suppression usage from `xiuxian-daochang` by converting no-op instance methods into
associated/static helpers.

Targets:

- `packages/rust/crates/xiuxian-daochang/src/agent/mcp.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/conversation.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/merge/discord.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/merge/telegram.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/channel/auth.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/parsing.rs`

## Changes Implemented

### 1) Removed unused-self wrapper in MCP soft-fail path

Actions:

- Converted `Agent::soft_fail_mcp_tool_error_output` into an associated helper
  without `&self`.
- Updated turn execution call site to `Self::soft_fail_mcp_tool_error_output`.

### 2) Removed unused-self suppressions in config merge layer

Actions:

- Converted principal-merge implementations to associated helpers:
  - `DiscordAclPrincipalSettings::merge(base, overlay)`
  - `TelegramAclPrincipalSettings::merge(base, overlay)`
- Updated option-merge call sites to use explicit type-qualified calls.

### 3) Removed unused-self in Discord identity normalization path

Actions:

- Converted `DiscordChannel::normalize_identity` to an associated helper.
- Converted ACL identity builder to associated helper as well
  (`build_acl_identities`), then updated parsing call sites.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "allow\\(clippy::unused_self" \
  packages/rust/crates/xiuxian-daochang/src \
  packages/rust/crates/xiuxian-daochang/tests
```

Results:

- `cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic`: pass for
  `xiuxian-daochang` (only existing `xiuxian-zhixing` warnings remain in workspace).
- `allow(clippy::unused_self)` is absent in `xiuxian-daochang/src` and
  `xiuxian-daochang/tests`.

Additional validation note:

- `cargo test -p xiuxian-daochang --lib` is currently blocked by unrelated ongoing
  `xiuxian-qianhuan` compilation breakage (`manifestation::templates` import /
  module resolution), not by this cleanup wave.

## Outcome

- `unused_self` suppression debt in `xiuxian-daochang` is reduced to zero.
- Call boundaries are clearer: helpers that do not need instance state are now
  explicit associated functions, improving API intent and pedantic cleanliness.
