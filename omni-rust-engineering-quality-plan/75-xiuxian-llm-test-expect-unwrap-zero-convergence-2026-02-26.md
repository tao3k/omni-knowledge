# Xiuxian-LLM Test `expect/unwrap` Zero-Convergence (2026-02-26)

## Scope

Complete the remaining `xiuxian-llm` test-lane cleanup by removing all
file-level `clippy::expect_used` / `clippy::unwrap_used` suppressions and
replacing panic-style paths with `Result`-first error flow.

## Why

After the previous reduction wave, several test modules still carried
`expect/unwrap` suppression flags, and two files still contained
`expect`/`panic` style setup paths. This weakened strict clippy signal quality.

## Implemented Changes

1. Migrated MCP reconnect tests to explicit `Result` flow in:
   - `tests/mcp_pool_reconnect.rs`
2. Removed stale `clippy::expect_used` / `clippy::unwrap_used` suppressions in:
   - `tests/mcp_pool_core.rs`
   - `tests/mcp_transport_error.rs`
   - `tests/mcp_pool.rs`
   - `tests/llm_backend.rs`
   - `tests/mcp_pool_utils.rs`
   - `tests/mcp_health.rs`
   - `tests/mcp_facade.rs`
   - `tests/embedding_backend.rs`
   - `tests/mcp_wait_heartbeat.rs`
3. Replaced setup-time `panic` / `expect` paths with `Result` propagation in:
   - `tests/embedding_openai_compat.rs`
   - `tests/mistral_runtime.rs`
4. Verified no remaining `expect/unwrap` suppression markers in
   `xiuxian-llm/tests` via `rg`.

## Verification Evidence

Executed:

```bash
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-llm/tests
cargo fmt -p xiuxian-llm
cargo test -p xiuxian-llm --test embedding_openai_compat
cargo test -p xiuxian-llm --test mistral_runtime
cargo test -p xiuxian-llm --test mcp_pool_reconnect
cargo test -p xiuxian-llm --test mcp_pool_core
cargo test -p xiuxian-llm --test mcp_pool_utils
cargo test -p xiuxian-llm --test mcp_health
cargo test -p xiuxian-llm --test mcp_wait_heartbeat
cargo test -p xiuxian-llm --test llm_backend
cargo test -p xiuxian-llm --test embedding_backend
cargo test -p xiuxian-llm --test mcp_facade
cargo test -p xiuxian-llm --test mcp_transport_error
cargo test -p xiuxian-llm --test mcp_pool
cargo clippy -p xiuxian-llm --all-targets -- -W clippy::pedantic
```

Result:

- `rg` returned no matches for `clippy::expect_used|clippy::unwrap_used` in
  `xiuxian-llm/tests`.
- All targeted test binaries passed.
- `clippy::pedantic` stayed green for `xiuxian-llm`.

## Outcome

`xiuxian-llm` test lanes now converge to zero `expect/unwrap` suppression
markers, with stronger strict-gate trustworthiness and clearer failure
semantics in runtime-heavy MCP integration tests.
