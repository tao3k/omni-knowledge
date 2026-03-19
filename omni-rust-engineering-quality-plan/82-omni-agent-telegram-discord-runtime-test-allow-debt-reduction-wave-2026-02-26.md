# 修仙道场 (Xiuxian Daochang) Telegram/Discord Runtime Test Allow-Debt Reduction Wave (2026-02-26)

## Scope

Reduce stale `expect/unwrap` suppression debt in runtime-heavy Telegram and
Discord test modules, and fix one newly exposed `xiuxian-daochang` pedantic warning
at source level.

## Why

Runtime command/session tests provide broad behavior coverage for channel
orchestration. Keeping obsolete `expect_used`/`unwrap_used` file-level flags
hides quality signal and slows convergence to strict test hygiene.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` in:
   - `tests/telegram_runtime/session_control_admin.rs`
   - `tests/telegram_runtime/session_reset.rs`
   - `tests/telegram_runtime/session_partition.rs`
   - `tests/telegram_runtime/session_stop.rs`
   - `tests/telegram_runtime/session_budget.rs`
   - `tests/telegram_runtime/transport_command_flow.rs`
   - `tests/telegram_runtime/partition_modes.rs`
   - `tests/telegram_runtime/session_admin.rs`
   - `tests/telegram_runtime/session_resume_flow.rs`
   - `tests/telegram_runtime/session_jobs.rs`
   - `tests/telegram_runtime/jobs_logging.rs`
   - `tests/discord_runtime/authorization.rs`
   - `tests/discord_runtime/managed_commands.rs`
   - `tests/discord_runtime/mod.rs`
   - `tests/discord_runtime/support.rs`
   - `tests/discord_runtime/session_preemption.rs`
2. Fixed newly exposed pedantic warning in:
   - `src/agent/bootstrap/tests.rs`
   - replaced `\"\".to_string()` with `String::new()` (`manual_string_new`).

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --test telegram_runtime_config
cargo test -p xiuxian-daochang --test channels_discord
cargo test -p xiuxian-daochang --test channels_discord_slash_authorization
cargo test -p xiuxian-daochang --test channels_discord_ingress
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- Targeted Telegram/Discord runtime tests passed.
- `xiuxian-daochang` pedantic clippy remained green after source warning fix.
- Non-blocking warnings still exist in sibling crates (`xiuxian-wendao`,
  `xiuxian-zhixing`), not in `xiuxian-daochang`.
- `xiuxian-daochang/tests` allow-marker file count dropped from `92` to `76`.

## Outcome

Runtime test lanes now have substantially lower stale suppression debt, and the
`xiuxian-daochang` crate itself remains pedantic-clean for this convergence slice.
