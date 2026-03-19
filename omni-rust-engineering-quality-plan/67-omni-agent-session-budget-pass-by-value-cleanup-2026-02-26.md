# 修仙道场 (Xiuxian Daochang) Session-Budget Pass-by-Value Cleanup (2026-02-26)

## Scope

This shard records a focused cleanup wave in `xiuxian-daochang` to remove
`clippy::large_types_passed_by_value` suppressions in session-budget response
formatting paths.

Targets:

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/replies/session_budget.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_context.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/replies/budget/dashboard.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/replies/budget/json.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/replies/budget/class_format.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/handlers/command_dispatch/session/budget.rs`

## Changes Implemented

### 1) Budget formatters now borrow snapshots instead of moving them

Actions:

- Changed snapshot formatters from value parameters to borrowed parameters:
  - Telegram:
    - `format_context_budget_snapshot(&SessionContextBudgetSnapshot)`
    - `format_context_budget_snapshot_json(&SessionContextBudgetSnapshot)`
  - Discord:
    - `format_context_budget_snapshot(&SessionContextBudgetSnapshot)`
    - `format_context_budget_snapshot_json(&SessionContextBudgetSnapshot)`
- Updated helper JSON formatters to take borrowed class snapshots:
  - Telegram: `format_context_budget_class_json(&SessionContextBudgetClassSnapshot)`
  - Discord: `format_context_budget_class_json(&SessionContextBudgetClassSnapshot)`

### 2) Command handler call-sites switched to borrowed passing

Actions:

- Telegram session command handler now passes `&snapshot` to budget formatters.
- Discord managed session-budget handler now passes `&snapshot` to budget
  formatters.

### 3) Suppression attributes removed

Actions:

- Deleted all four `#[allow(clippy::large_types_passed_by_value)]` attributes
  from the touched budget reply files.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
cargo test -p xiuxian-daochang --lib session_budget
rg -o "allow\\(clippy::[a-z0-9_]+(?:, clippy::[a-z0-9_]+)*\\)" \
  packages/rust/crates/xiuxian-daochang/src \
  packages/rust/crates/xiuxian-daochang/tests \
| sed -E 's/.*allow\\(//; s/\\)//' | tr ',' '\\n' \
| sed -E 's/^\\s*clippy:://; s/^\\s+|\\s+$//g' | sort | uniq -c | sort -nr
```

Results:

- `cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic`: pass for
  `xiuxian-daochang` (workspace still shows existing `xiuxian-zhixing` warning
  outside this scope).
- `cargo test -p xiuxian-daochang --lib session_budget`: pass (`5 passed`, `0 failed`).
- `large_types_passed_by_value` suppression category is now fully removed from
  `xiuxian-daochang`.
- Updated suppression inventory in `xiuxian-daochang`:
  - `wildcard_imports`: 11
  - `struct_field_names`: 3
  - `cast_precision_loss`: 3
  - `similar_names`: 2
  - `too_many_arguments`: 1
  - `cast_sign_loss`: 1
  - `cast_possible_truncation`: 1

## Outcome

- Session-budget formatting and command-dispatch paths now avoid moving large
  snapshot payloads.
- `xiuxian-daochang` suppression debt is reduced again with behavior-preserving,
  low-risk signature refactors.
