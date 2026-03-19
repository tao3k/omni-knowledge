# 修仙道场 (Xiuxian Daochang) Manual-Let-Else Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::manual_let_else` file-level allow markers and fixing all surfaced
real warnings with source-level rewrites.

## Implemented Changes

1. Removed `clippy::manual_let_else` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Rewrote surfaced `match` extraction patterns to idiomatic
   `let ... else` in test lanes, including:
   - memory and reflection tests:
     - `tests/agent/memory/recall_credit.rs`
     - `tests/agent/memory_recall_state.rs`
     - `tests/agent/memory_stream_consumer.rs`
     - `tests/agent/reflection.rs`
   - gateway, MCP, session and runtime tests:
     - `tests/gateway/http/runtime.rs`
     - `tests/gateway_validation.rs`
     - `tests/mcp_health_gate.rs`
     - `tests/mcp_pool_hard_timeout.rs`
     - `tests/mcp_connect_startup.rs`
     - `tests/mcp_discover_cache.rs`
     - `tests/session_redis.rs`
   - channel/ACL and embedding tests:
     - `tests/channels_control_command_authorization.rs`
     - `tests/channels_discord.rs`
     - `tests/channels_discord_ingress.rs`
     - `tests/discord_acl_overrides.rs`
     - `tests/telegram_acl_overrides.rs`
     - `tests/embedding_client_cache.rs`
   - job/session support tests:
     - `tests/jobs_manager.rs`
     - `tests/jobs_scheduler.rs`
     - `tests/agent_session_context.rs`
     - `tests/agent_memory_scope_isolation.rs`
     - `tests/agent/system_prompt_injection_state.rs`
     - `tests/test_support_parsers.rs`
     - `tests/agent_injection.rs`
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::manual_let_else" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::manual_let_else` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::manual_let_else` with strict
pedantic and `too_many_lines` validation still green.
