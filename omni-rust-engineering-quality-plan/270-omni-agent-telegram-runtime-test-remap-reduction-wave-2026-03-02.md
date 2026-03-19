# 270. xiuxian-daochang telegram runtime Test Remap Reduction Wave (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - reduce `channels_telegram_runtime_unit` source remap usage by replacing
    managed-command and telegram-command dependencies with stable boundaries,
  - keep runtime behavior coverage green,
  - record the remaining remap blocker explicitly.

## Changes

### 1) Replaced managed-command and command-parser remaps with stable boundaries

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_runtime_unit.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/telegram_parser.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/types.rs`

Actions:

- removed remaps to:
  - `src/channels/managed_commands/mod.rs`,
  - `src/channels/telegram/commands.rs`,
- switched harness to:
  - `xiuxian_daochang::test_support::{detect_managed_control_command, detect_managed_slash_command, ManagedControlCommand, ManagedSlashCommand, ...telegram parser exports...}`,
- added missing `is_stop_command` test-support export,
- added missing command methods in test-support enums used by included runtime
  modules:
  - `ManagedSlashCommand::{scope, canonical_command}`,
  - `ManagedControlCommand::canonical_command`.

### 2) Kept one remap intentionally for runtime-config type coupling

Current remaining remap in this harness:

- `#[path = "../src/channels/telegram/runtime_config.rs"]`

Reason:

- replacing with `xiuxian_daochang::TelegramRuntimeConfig` introduced a type mismatch
  between:
  - library `ForegroundQueueMode` type,
  - harness-included `managed_runtime::queue_mode::ForegroundQueueMode` type.
- this single remap is retained temporarily until queue-mode type boundary is
  unified for this harness.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test channels_telegram_runtime_unit
```

Result:

- `74 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0), no warnings/errors.

### 3) Structural proof command

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/\" \
  packages/rust/crates/xiuxian-daochang/tests/channels_telegram_runtime_unit.rs
```

Result:

- one remaining match:
  - `#[path = "../src/channels/telegram/runtime_config.rs"]`.

## Outcome

- telegram runtime harness removed two of three direct source remaps,
- runtime behavior coverage remains fully green (`74` tests),
- one bounded remap remains with explicit root cause and follow-up path.
