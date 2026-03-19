# 198. Xiuxian-LLM Test Allow-Debt Zero Convergence Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-llm`
- Focus:
  - test files carrying file-level `#![allow(...)]` attributes in
    `tests/` (16 files total)

## Why This Wave

`xiuxian-llm/tests` still carried broad file-level suppressions for
`missing_docs`, `doc_markdown`, `float_cmp`, `field_reassign_with_default`,
`manual_async_fn`, and `no_effect_underscore_binding`. The target was full
suppression removal with root-cause code fixes only.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from all affected test files:
   - `tests/mcp_pool_runtime.rs`
   - `tests/mcp_transport_error.rs`
   - `tests/mcp_pool.rs`
   - `tests/llm_backend.rs`
   - `tests/embedding_backend.rs`
   - `tests/mistral_sdk_embedding.rs`
   - `tests/mcp_health.rs`
   - `tests/mcp_pool_core.rs`
   - `tests/mcp_pool_utils.rs`
   - `tests/mcp_pool_retry.rs`
   - `tests/mcp_discover_cache.rs`
   - `tests/mcp_facade.rs`
   - `tests/mcp_wait_heartbeat.rs`
   - `tests/mistral_runtime.rs`
   - `tests/mcp_pool_hard_timeout.rs`
   - `tests/mcp_pool_reconnect.rs`

2. Added explicit module docs to test entry files to satisfy missing-doc lint.

3. Fixed clippy root causes surfaced after suppression removal:
   - `tests/mcp_facade.rs`:
     replaced no-effect underscore bindings with symbol-touch helper and
     explicit type assertions.
   - `tests/embedding_openai_compat.rs`:
     replaced manual `match` with `let ... else` pattern.
   - `tests/mcp_pool_hard_timeout.rs`:
     converted handler methods to `async fn` and removed redundant async blocks.
   - `tests/mcp_pool_utils.rs` and `tests/mcp_pool_reconnect.rs`:
     replaced strict float equality assertions with tolerance-based checks.
   - `tests/mcp_pool_core.rs`:
     replaced field reassignment-after-default with struct literal initialization.

No new suppressions were introduced.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-llm
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-llm cargo clippy -p xiuxian-llm --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass (no warnings)

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-llm cargo nextest run -p xiuxian-llm
```

- Result: pass
- Summary: `64 passed`, `0 failed`, `0 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-llm/tests -g '*.rs' | wc -l`
  - Before this wave: `16`
  - After this wave: `0`
  - Net reduction: `16` files

## Engineering Outcome

- `xiuxian-llm/tests` now has zero file-level `#![allow(...)]` debt.
- The MCP/runtime integration test lane remains fully green under strict
  pedantic validation without suppression fallback.
