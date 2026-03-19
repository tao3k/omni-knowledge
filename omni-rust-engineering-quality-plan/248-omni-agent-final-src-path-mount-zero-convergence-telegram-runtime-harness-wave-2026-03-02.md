# 248. xiuxian-daochang Final `src` Path-Mount Zero Convergence (Telegram Runtime Harness Wave, 2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Goal:
  - remove the remaining `src`-side test path mount in
    `channels::telegram::runtime`,
  - keep the same runtime test coverage through package-top harnesses,
  - revalidate the full migrated harness matrix plus mandatory clippy.

## Code Changes

### Removed remaining `src` mount

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/mod.rs`
  - removed:
    - `#[cfg(test)]`
    - `#[path = "../../../../tests/telegram_runtime/mod.rs"]`
    - `mod tests;`

### Added package-top harness for Telegram runtime

- `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_runtime_unit.rs`
  - hosts the Telegram runtime unit lane from package top,
  - reuses existing suite:
    - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/mod.rs`
    - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/*.rs`
  - introduces minimal harness-side shims for:
    - `agent` runtime behavior required by managed/runtime command lanes,
    - `gateway` embedding-route hooks,
    - webhook idempotency trait/store contracts.

### Include compatibility fix

- `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/mod.rs`
  - adjusted module doc comment style for include-based harness loading.

No broad `#[allow(...)]` suppression was introduced.

## Validation Evidence

### 1) Global residual scan (`src` path mounts)

```bash
rg --line-number --glob 'packages/rust/crates/*/src/**/*.rs' '#\[path\s*=\s*"[^"]*tests/[^"]*"\]' | sort
```

Result:

- no matches.

### 2) Telegram runtime migrated lane

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang --test channels_telegram_runtime_unit
```

Result:

- `74 passed`, `0 failed`.

### 3) Aggregated migrated harness matrix

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang --test agent_session_context_unit --test channels_discord_runtime_unit --test channels_telegram_runtime_unit
```

Result:

- `104 passed`, `0 failed`.

### 4) Mandatory touched-crate clippy

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- completed successfully (`exit code 0`).

## Delta Summary

- Previous global residual count: `6` (recorded in `247-...`).
- New global residual count: `0`.
- Net reduction for this convergence wave: `6` (session-context, discord runtime, telegram runtime lanes fully migrated to package-top harnesses).
