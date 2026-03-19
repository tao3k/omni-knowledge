# 271. xiuxian-daochang telegram runtime Test Remap Zero Convergence (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - close the final remap in `channels_telegram_runtime_unit`,
  - keep the harness compatible with local included runtime queue-mode types,
  - preserve full telegram runtime test coverage.

## Changes

### Replaced last `runtime_config` remap with local adapter type

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_runtime_unit.rs`

Actions:

- removed:
  - `#[path = "../src/channels/telegram/runtime_config.rs"]`,
- introduced test-local `channels::telegram::runtime_config::TelegramRuntimeConfig`
  adapter with:
  - same field surface used by included telegram runtime modules,
  - `from_env` and `from_lookup_for_test` constructors,
  - conversion from `xiuxian_daochang::TelegramRuntimeConfig` into harness-local
    `ForegroundQueueMode` via canonical mode string mapping.

Rationale:

- this avoids the previous queue-mode type mismatch between:
  - library `xiuxian_daochang::ForegroundQueueMode`,
  - harness-local `channels::managed_runtime::ForegroundQueueMode`.

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

- no matches.

## Outcome

- `channels_telegram_runtime_unit` reached zero `#[path = "../src/..."]` remaps,
- telegram runtime integration coverage remains fully green,
- `xiuxian-daochang/tests` remap debt now concentrates only in `tests/llm.rs`.
