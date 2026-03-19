# 290. xiuxian-daochang telegram-runtime test remap elimination and suite zero-remap convergence (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target lane: `tests/channels_telegram_runtime_unit.rs`
- Goal: remove all remaining Telegram runtime source includes and close the
  `xiuxian-daochang/tests` remap debt to zero.

## Implementation

1. Added Telegram runtime test-support bridge:
   - `src/test_support/telegram_runtime.rs`
   - Exposed stable wrappers for:
     - foreground interrupt token control
     - `process_update` runtime entry
     - snapshot interval resolution
2. Wired test-support exports:
   - `src/test_support/mod.rs`
3. Added crate-internal Telegram runtime test bridge points:
   - `src/channels/telegram/runtime/mod.rs`
   - `src/channels/telegram/runtime/dispatch/interrupt.rs`
4. Migrated Telegram runtime test imports to public crate/test-support paths:
   - `tests/channels/telegram/runtime/tests/*.rs`
5. Replaced top-level harness with package-top entrypoint:
   - `tests/channels_telegram_runtime_unit.rs`
   - includes `#![recursion_limit = "256"]` for macro expansion stability.

## Verification

- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test channels_telegram_runtime_unit --test channels_discord_runtime_unit --test embedding`
  - result: `110 passed`, `0 skipped`, `0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass (with one existing warning debt in `src/agent/mcp.rs`)
- Remap debt counter:
  - `rg -n "include!\(\"\.\./src/|#\[path\s*=\s*\"\.\./src/|#\[path\s*=\s*\"\.\./\.\./src/" packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l`
  - result: `9 -> 0`

## Open Debt

- `clippy::too_many_lines` remains in:
  - `packages/rust/crates/xiuxian-daochang/src/agent/mcp.rs:67`
  - `call_mcp_tool_with_diagnostics` (`132` lines vs threshold `100`)
