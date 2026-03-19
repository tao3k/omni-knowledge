# 修仙道场 (Xiuxian Daochang) Unnecessary-Literal-Bound Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::unnecessary_literal_bound` file-level allow markers and fixing
surfaced signature-level warnings.

## Implemented Changes

1. Removed `clippy::unnecessary_literal_bound` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced warnings by updating mock channel `name()` signatures from
   `&str` to `&'static str` where implementations return string literals:
   - `tests/discord_runtime/support.rs`
   - `tests/telegram_runtime/mod.rs`
   - `tests/telegram_runtime/session_admin.rs`
   - `tests/telegram_runtime/session_control_admin.rs`
   - `tests/telegram_runtime/session_feedback.rs`
   - `tests/telegram_runtime/session_help.rs`
   - `tests/telegram_runtime/session_memory.rs`
   - `tests/telegram_runtime/session_partition.rs`
   - `tests/telegram_runtime/session_slash_acl.rs`
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::unnecessary_literal_bound" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::unnecessary_literal_bound` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::unnecessary_literal_bound` with strict
pedantic and `too_many_lines` validation still green.
