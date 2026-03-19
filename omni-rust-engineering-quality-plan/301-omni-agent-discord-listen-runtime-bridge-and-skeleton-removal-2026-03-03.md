# 301. xiuxian-daochang Discord listen runtime bridge and skeleton removal (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target area:
  - `src/channels/discord/channel/trait_impl.rs`
  - `src/channels/discord/runtime/gateway/mod.rs`
  - `src/channels/discord/channel/state.rs`
  - `tests/channels_discord.rs`
- Goal: remove hardcoded `listen` skeleton failure and wire Discord `Channel::listen`
  to the real serenity gateway listener path.

## Implementation

1. Replaced hardcoded `listen` bailout with runtime bridge:
   - `DiscordChannel::listen` now calls
     `runtime::run_discord_gateway_listener(Arc::new(self.clone()), tx)`.
2. Added runtime listener entrypoint:
   - Introduced `run_discord_gateway_listener(channel, tx)` in
     `runtime/gateway/mod.rs`.
   - This function validates token, builds serenity client, installs the
     existing gateway event handler, and starts the gateway loop.
3. Added deterministic token guard:
   - Added `ensure!(!bot_token.trim().is_empty(), "discord bot token cannot be empty")`
     to both runtime gateway paths (`run_discord_gateway_listener` and
     `run_discord_gateway`).
4. Enabled safe channel cloning for listener bootstrap:
   - Implemented `Clone` for `DiscordChannel` by snapshotting lock-protected
     runtime state (`session_partition`, recipient overrides, sender identity
     cache) and cloning immutable transport/policy fields.
5. Removed stale “skeleton” wording:
   - Updated Discord module/channel/constructor docs to reflect active runtime.
6. Updated regression test contract:
   - Replaced old `not implemented` assertion with deterministic empty-token
     validation (`discord_listen_rejects_empty_bot_token`).

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Targeted Discord regression:
  - `cargo nextest run -p xiuxian-daochang --test channels_discord --test channels_discord_runtime_unit --test channels_discord_parsing --test channels_discord_ingress --test channels_discord_send --test channels_discord_slash_authorization`
  - result: `68 passed`, `0 skipped`, `0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass (existing unrelated warnings remain in other lanes:
    `xiuxian-llm` `match_same_arms`, `xiuxian-daochang` `litellm_ocr.rs`
    `too_many_lines`, `llm/test_api.rs` doc markdown)

## Outcome

- Discord `Channel::listen` is no longer a guaranteed runtime self-termination
  point; it now boots the real gateway listener path.
- Runtime boundary is explicit (`run_discord_gateway_listener`) and reusable,
  while preserving current gateway event handling and queue semantics.
