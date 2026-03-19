# 269. xiuxian-daochang discord runtime Test Remap Elimination (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - remove remaining `#[path = "../src/..."]` remaps in
    `channels_discord_runtime_unit`,
  - keep runtime behavior coverage intact,
  - validate touched crate under nextest and mandatory clippy gates.

## Changes

### 1) Removed direct source remaps from Discord runtime harness

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/channels_discord_runtime_unit.rs`

Actions:

- removed remaps to:
  - `src/channels/traits.rs`,
  - `src/channels/managed_commands/mod.rs`,
  - `src/channels/discord/session_partition.rs`,
- replaced trait and partition dependencies with stable public boundaries:
  - `xiuxian_daochang::{Channel, ChannelAttachment, ChannelMessage, DiscordSessionPartition, RecipientCommandAdminUsersMutation}`,
- replaced managed-command dependency with test-support boundary:
  - `xiuxian_daochang::test_support::{detect_managed_control_command, detect_managed_slash_command, ManagedControlCommand, ManagedSlashCommand}`,
- defined slash-scope constants locally in the harness to preserve prior
  assertion surface.

### 2) Filled missing managed-command methods in test-support types

Updated:

- `packages/rust/crates/xiuxian-daochang/src/test_support/types.rs`

Actions:

- added `ManagedSlashCommand::{scope, canonical_command}`,
- added `ManagedControlCommand::canonical_command`,
- aligned command/scope strings with production managed-command contract so
  included Discord runtime modules compile and execute without source remaps.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test channels_discord_runtime_unit
```

Result:

- `28 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0), no warnings/errors.

### 3) Structural proof command

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/\" \
  packages/rust/crates/xiuxian-daochang/tests/channels_discord_runtime_unit.rs
```

Result:

- no matches.

## Outcome

- Discord runtime integration harness no longer depends on source remapping for
  traits/managed-commands/session-partition modules,
- runtime behavior coverage remains intact (`28` tests green),
- touched crate remains clean under required clippy gate.
