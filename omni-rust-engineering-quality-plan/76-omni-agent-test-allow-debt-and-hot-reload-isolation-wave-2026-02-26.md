# 修仙道场 (Xiuxian Daochang) Test Allow-Debt and Hot-Reload Isolation Wave (2026-02-26)

## Scope

Continue `xiuxian-daochang` test-lane convergence by:

1. removing stale `expect_used`/`unwrap_used` suppressions in a focused batch,
2. fixing clippy pedantic warnings in hot-reload source/test paths,
3. removing environment-variable cross-test contamination in hot-reload tests.

## Why

`xiuxian-daochang/tests` still had broad file-level allow debt. During verification,
`agent_suite` exposed a real flake risk: hot-reload bootstrap test mutated env
variables in-process and leaked behavior into concurrent tests.

## Implemented Changes

1. Removed `clippy::expect_used` / `clippy::unwrap_used` from:
   - `tests/agent_suite.rs`
   - `tests/agent_summary.rs`
   - `tests/contracts.rs`
   - `tests/llm/backend.rs`
   - `tests/llm/provider_mode.rs`
   - `tests/session_summary.rs`
   - `tests/valkey_url_precedence.rs`
   - `tests/observability_session_events.rs`
2. Fixed `clippy::map_unwrap_or` in:
   - `src/agent/bootstrap/hot_reload.rs`
3. Refactored hot-reload bootstrap test isolation:
   - `tests/agent/bootstrap_hot_reload.rs`
   - Replaced in-process env mutation + lock-based guarding with child-process
     probe execution (`--exact`), so env overrides are process-scoped and do
     not leak across concurrent tests.
4. Kept `AgentConfig` initialization idiomatic in the hot-reload test via
   struct literal + `..Default::default()` (no reassign-after-default pattern).

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --test agent_suite
cargo test -p xiuxian-daochang --test agent_summary
cargo test -p xiuxian-daochang --test contracts
cargo test -p xiuxian-daochang --test session_summary
cargo test -p xiuxian-daochang --test valkey_url_precedence
cargo test -p xiuxian-daochang --test observability_session_events
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- All targeted tests passed.
- `clippy::pedantic` for `xiuxian-daochang` stayed green after fixes.
- `xiuxian-daochang/tests` allow-marker file count dropped from `131` to `123`.

## Outcome

`xiuxian-daochang` test quality improved in two dimensions:

1. lower suppression debt in high-signal test targets,
2. deterministic hot-reload bootstrap behavior under parallel test execution.
