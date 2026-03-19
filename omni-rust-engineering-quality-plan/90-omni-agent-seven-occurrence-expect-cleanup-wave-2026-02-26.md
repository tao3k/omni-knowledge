# 修仙道场 (Xiuxian Daochang) Seven-Occurrence Expect Cleanup Wave (2026-02-26)

## Scope

Continue suppression-debt convergence by cleaning the seven-occurrence queue in
`xiuxian-daochang` tests.

## Why

Queue-ordered cleanup remains the most reliable way to reduce debt while
keeping behavior stable and verification repeatable.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced panic-style extraction with explicit handling in:
   - `tests/agent_memory_scope_isolation.rs`
   - `tests/mcp_discover_cache.rs`
   - `tests/mcp_pool_reconnect.rs`
2. Fixed newly surfaced compile-time macro boundary issue discovered during
   full clippy run:
   - `src/agent/bootstrap/zhixing.rs`
   - replaced invalid `embedded_utf8_files!(BUILTIN_ZHIXING_TEMPLATE_DIR)` call
     with direct `collect_embedded_utf8_files(&BUILTIN_ZHIXING_TEMPLATE_DIR)`.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- `xiuxian-daochang` remains buildable under pedantic clippy.
- marker-file count in `xiuxian-daochang/tests` dropped from `19` to `16`.
- existing workspace sibling warnings remained in:
  - `xiuxian-wendao`
  - `xiuxian-zhixing`
  - `xiuxian-qianhuan`
- additional non-blocking warning observed in `xiuxian-daochang`:
  `clippy::too_many_lines` at `src/agent/bootstrap/zhixing.rs`.

## Outcome

The remaining test-lane suppression queue now starts at eight-occurrence files.

## Next Queue

Prioritize eight-occurrence files:

- `tests/discord_acl_overrides.rs`
- `tests/embedding_client_cache.rs`
- `tests/telegram_acl_overrides.rs`
