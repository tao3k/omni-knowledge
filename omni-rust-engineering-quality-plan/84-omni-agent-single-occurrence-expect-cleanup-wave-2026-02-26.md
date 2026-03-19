# 修仙道场 (Xiuxian Daochang) Single-Occurrence Expect Cleanup Wave (2026-02-26)

## Scope

Continue `xiuxian-daochang` test-lane suppression-debt convergence by fixing the
next batch of files that each had exactly one remaining
`expect`/`expect_err`/`unwrap` site.

## Why

This batch provides high signal-to-effort progress: each file can become
`expect`-free with one root-cause fix, and stale file-level
`clippy::expect_used` / `clippy::unwrap_used` can then be removed.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced the single panic-style extraction with explicit handling in:
   - `tests/agent/memory_recall.rs`
   - `tests/channels_control_command_authorization.rs`
   - `tests/channels_telegram_send_gate.rs`
   - `tests/multiple_mcp.rs`
   - `tests/telegram_runtime/session_feedback.rs`
   - `tests/telegram_runtime/session_injection.rs`
   - `tests/telegram_runtime/session_memory.rs`
   - `tests/telegram_runtime/session_preemption.rs`
   - `tests/telegram_runtime/session_slash_acl.rs`
   - `tests/telegram_runtime/webhook_security.rs`
2. Preferred explicit branch handling and `Result` propagation:
   - `Option` extraction moved to `let Some(...) = ... else { ... }`.
   - JSON field extraction moved to `ok_or_else(anyhow!(...))?`.
   - expected-failure checks moved from `expect_err(...)` to `match` + explicit
     `bail!(...)` on unexpected success.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --test agent_suite --test multiple_mcp --test channels_control_command_authorization --test channels_telegram_send_gate --no-run
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- Targeted test binaries compiled successfully.
- Full `xiuxian-daochang` test target compile (`--tests --no-run`) succeeded.
- `xiuxian-daochang` remained green under pedantic clippy.
- Workspace-level pedantic warnings still exist in sibling crates
  (`xiuxian-wendao`, `xiuxian-zhixing`), unchanged by this wave.
- `xiuxian-daochang/tests` allow-marker file count dropped from `56` to `46`.

## Outcome

This wave removed another full slice of stale suppression markers without
adding new ignores, and continued the root-cause-first convergence path for
test reliability and lint signal quality.

## Next Queue

Prioritize files currently at `2` occurrences each to keep fast convergence:

- `tests/agent/memory_stream_consumer.rs`
- `tests/agent/system_prompt_injection_state.rs`
- `tests/agent_context_budget.rs`
- `tests/agent_context_window_recovery.rs`
- `tests/agent_integration.rs`
- `tests/channels_discord.rs`
- `tests/channels_discord_slash_authorization.rs`
- `tests/contracts/test_runtime_contracts.rs`
- `tests/llm/http_request.rs`
- `tests/mcp_health_gate.rs`
- `tests/shortcuts.rs`
