# 修仙道场 (Xiuxian Daochang) Two-Occurrence Expect Cleanup Wave (2026-02-26)

## Scope

Continue `xiuxian-daochang` test-lane suppression-debt convergence by cleaning the
next prioritized queue where each target file had exactly two remaining
`expect`/`expect_err`/`unwrap` occurrences.

## Why

After the single-occurrence wave, the two-occurrence queue was the highest
throughput lane for steady marker reduction without introducing broad behavior
changes.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced panic-style extraction with explicit handling in:
   - `tests/shortcuts.rs`
   - `tests/mcp_health_gate.rs`
   - `tests/llm/http_request.rs`
   - `tests/contracts/test_runtime_contracts.rs`
   - `tests/channels_discord_slash_authorization.rs`
   - `tests/channels_discord.rs`
   - `tests/agent_context_budget.rs`
   - `tests/agent/system_prompt_injection_state.rs`
   - `tests/agent_context_window_recovery.rs`
   - `tests/agent/memory_stream_consumer.rs`
   - `tests/agent_integration.rs`
2. Applied root-cause conversion patterns:
   - `expect`/`unwrap` -> explicit `match` branches with clear failure messages.
   - `expect_err` -> explicit `match` with rejection of unexpected success.
   - `Option` extraction -> `let Some(...) = ... else { ... }`.
   - `Result` tests -> explicit error propagation where appropriate.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- `xiuxian-daochang` test targets compiled successfully.
- `xiuxian-daochang` stayed green under pedantic clippy.
- Workspace-level pedantic warnings remained only in sibling crates
  (`xiuxian-wendao`, `xiuxian-zhixing`), unchanged by this wave.
- `xiuxian-daochang/tests` allow-marker file count dropped from `46` to `35`.

## Outcome

The queue-based convergence remains stable: another 11-file batch is now
`expect/unwrap` suppression-free with explicit failure semantics and no new
lint suppressions.

## Next Queue

Prioritize `3`-occurrence files next:

- `tests/agent/graph_planner.rs`
- `tests/discover_cache_valkey_precedence.rs`
- `tests/gateway_validation.rs`
- `tests/jobs_scheduler.rs`
