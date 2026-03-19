# 289. xiuxian-daochang discord-runtime test remap elimination via test-support runtime bridge (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target lane: `tests/channels_discord_runtime_unit.rs`
- Goal: remove all Discord runtime source-include remaps and converge to
  package-top integration tests through stable `test_support` boundaries.

## Implementation

1. Added Discord runtime test-support bridge:
   - `src/test_support/discord_runtime.rs`
   - Exposed:
     - `DiscordForegroundInterruptController` (public wrapper)
     - `process_discord_message_with_interrupt(...)`
     - `resolve_discord_snapshot_interval_secs(...)`
2. Wired exports:
   - `src/test_support/mod.rs`
3. Added crate-internal Discord runtime test bridge points:
   - `src/channels/discord/runtime/mod.rs`
   - `test_process_discord_message_with_interrupt(...)`
   - `test_resolve_snapshot_interval_secs(...)`
4. Visibility alignment for crate-internal test bridge access:
   - `src/channels/mod.rs`: `discord` -> `pub(crate) mod discord`
   - `src/channels/discord/mod.rs`: `runtime` -> `pub(crate) mod runtime`
   - `src/channels/discord/runtime/interrupt.rs`:
     `ForegroundInterruptController` + methods -> `pub(crate)`
5. Migrated Discord runtime test imports to public crate surface / test-support:
   - `tests/channels/discord/runtime/tests/support.rs`
   - `tests/channels/discord/runtime/tests/managed_commands.rs`
   - `tests/channels/discord/runtime/tests/authorization.rs`
   - `tests/channels/discord/runtime/tests/session_preemption.rs`
   - `tests/channels/discord/runtime/tests/logging.rs`
   - `tests/channels/discord/runtime/tests/telemetry.rs`
6. Replaced top-level harness with standard package-top entrypoint:
   - `tests/channels_discord_runtime_unit.rs`
   - now only mounts `tests/channels/discord/runtime/tests/mod.rs`.
7. Lint follow-up:
   - Added `#[must_use]` on `DiscordForegroundInterruptController::begin_generation`.

## Verification

- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test channels_discord_runtime_unit --test embedding`
  - result: `36 passed`, `0 skipped`, `0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass (with one existing warning debt in `src/agent/mcp.rs`)
- Remap debt counter:
  - `rg -n "include!\(\"\.\./src/|#\[path\s*=\s*\"\.\./src/|#\[path\s*=\s*\"\.\./\.\./src/" packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l`
  - result: `15 -> 9`

## Open Debt

- `clippy::too_many_lines` remains in:
  - `packages/rust/crates/xiuxian-daochang/src/agent/mcp.rs:67`
  - `call_mcp_tool_with_diagnostics` (`132` lines vs threshold `100`)
- Planned next action: split this function into focused helpers in a dedicated
  modularization wave, without adding suppression.
