# 修仙道场 (Xiuxian Daochang) Struct Field Names De-Legacy Convergence (2026-02-26)

## Scope

Complete the final `struct_field_names` convergence in `xiuxian-daochang` by
removing legacy-compatible field naming and migrating directly to concise,
domain-focused names.

## Why

The previous state still carried compatibility-oriented field names and
`#[allow(clippy::struct_field_names)]` in production source. This blocked
full pedantic convergence and kept naming debt in core runtime/config types.

## Implemented Changes

1. Removed all remaining production `struct_field_names` suppressions in
   `xiuxian-daochang/src`.
2. Renamed `McpSettings` fields from `agent_*` to concise names:
   `pool_size`, `handshake_timeout_secs`, `connect_retries`,
   `strict_startup`, `connect_retry_backoff_ms`, `tool_timeout_secs`,
   `list_tools_cache_ttl_ms`, `discover_cache_enabled`,
   `discover_cache_key_prefix`, `discover_cache_ttl_secs`.
3. Updated all runtime/config merge and consumer call sites to use the new
   `McpSettings` names.
4. Updated config/test YAML keys to the new `mcp.*` names (no alias fallback).
5. Renamed Telegram/Discord slash policy fields:
   `global`, `session_status`, `session_budget`, `session_memory`,
   `session_feedback`, `job_status`, `jobs_summary`, `background_submit`.
6. Updated all policy builders, runtime node wiring, and slash authorization
   tests to the new names while keeping ACL-override parsing structures
   unchanged where they are separate from policy model naming.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
cargo test -p xiuxian-daochang --lib
cargo test -p xiuxian-daochang --test config_settings
cargo test -p xiuxian-daochang --test discover_cache_valkey_precedence
cargo test -p xiuxian-daochang --test channels_telegram_slash_authorization
cargo test -p xiuxian-daochang --test channels_discord_slash_authorization
cargo test -p xiuxian-daochang --test telegram_acl_overrides
cargo test -p xiuxian-daochang --test discord_acl_overrides
```

Result:

- `clippy` passed for `xiuxian-daochang` with pedantic enabled.
- Library test suite passed (`224 passed; 0 failed; 8 ignored`).
- All targeted config/discover/slash/ACL integration tests passed.

## Outcome

`xiuxian-daochang` production source now has zero `struct_field_names` suppression
attributes, with direct (non-legacy) naming convergence and validated behavior.
