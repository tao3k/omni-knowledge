# Xiuxian-LLM Test Allow-Debt Reduction Wave (2026-02-26)

## Scope

Continue strict test-quality convergence in `xiuxian-llm` by reducing
file-level clippy suppression debt and replacing panic-prone patterns with
`Result`-first test flow.

## Why

Several `xiuxian-llm` test modules still carried broad file-level
`#![allow(...)]` lists including `clippy::expect_used` and
`clippy::unwrap_used`.

This blocks trust in strict-gate signal quality and allows regressions to hide
in high-impact MCP runtime tests.

## Implemented Changes

1. Migrated `expect`/`unwrap` in the following tests to explicit
   `Result`-based handling:
   - `tests/mcp_discover_cache.rs`
   - `tests/mcp_pool_retry.rs`
   - `tests/mcp_pool_runtime.rs`
2. Removed `clippy::expect_used` and `clippy::unwrap_used` from file-level
   allow lists in those files.
3. Updated helper functions in runtime tests to propagate IO/bootstrap errors
   with `?` (`reserve_local_addr`, `spawn_mock_server`).
4. Fixed server-task shutdown semantics after `abort()`:
   - treat `JoinError::is_cancelled()` as expected,
   - only fail on non-cancelled join errors.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-llm
cargo test -p xiuxian-llm --test mcp_discover_cache
cargo test -p xiuxian-llm --test mcp_pool_retry
cargo test -p xiuxian-llm --test mcp_pool_runtime
cargo test -p xiuxian-llm --test llm_openai_client
cargo clippy -p xiuxian-llm --all-targets -- -W clippy::pedantic
```

Result:

- All targeted `xiuxian-llm` MCP/OpenAI client tests passed.
- Strict pedantic clippy lane remained green for `xiuxian-llm`.
- No new suppression attributes were introduced.

## Outcome

`xiuxian-llm` now has stronger pedantic-gate integrity in key MCP test lanes,
with reduced allow-debt and more explicit failure semantics.
