# 修仙道场 (Xiuxian Daochang) Marker Burndown Wave 2 and Cross-Crate Pedantic Cleanup (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` marker-debt reduction for
`clippy::too_many_lines`/`clippy::too_many_arguments`, and remove newly exposed
pedantic warnings in `xiuxian-daochang` and `xiuxian-wendao` without adding
suppressions.

## Implemented Changes

1. Removed stale `too_many_lines`/`too_many_arguments` markers from 8 additional
   `xiuxian-daochang` test modules by reducing each file to the lint threshold:
   - `packages/rust/crates/xiuxian-daochang/tests/mcp_health_gate.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_stop.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/graph_planner.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/discord_runtime/authorization.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/session_reset.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/omega_decision.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_chunking.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/system_prompt_injection_state.rs`
2. Fixed `xiuxian-wendao` pedantic warning
   (`case_sensitive_file_extension_comparisons`) in
   `packages/rust/crates/xiuxian-wendao/src/zhenfa_router/native/xml_lite.rs`
   by introducing case-insensitive extension detection via `Path::extension()`.
3. Fixed `xiuxian-daochang` pedantic warning (`too_many_lines`) in
   `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/zhixing.rs` by
   extracting reminder-notification worker startup into
   `spawn_reminder_notification_worker(...)`.
4. Fixed follow-up `unused_mut` after extraction in
   `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/zhixing.rs`.
5. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -o "clippy::too_many_lines|clippy::too_many_arguments" \
  packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' \
  | sed 's/.*://g' | sort | uniq -c | sort -nr
cargo fmt -p xiuxian-wendao -p xiuxian-daochang
cargo clippy -p xiuxian-wendao -- -W clippy::pedantic
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- Remaining marker counts in `xiuxian-daochang/tests`:
  - `too_many_lines`: `81`
  - `too_many_arguments`: `80`
- `cargo clippy -p xiuxian-wendao -- -W clippy::pedantic`: pass (`0` warnings).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`: pass (`0`).
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0` warnings).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0` warnings).

## Outcome

Wave 2 reduced `xiuxian-daochang/tests` marker debt by another 8 files while keeping
strict clippy lanes green and removing newly surfaced production-code pedantic
warnings in both `xiuxian-daochang` and `xiuxian-wendao`.
