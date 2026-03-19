# 修仙道场 (Xiuxian Daochang) Six-Occurrence Expect Cleanup Wave (2026-02-26)

## Scope

Continue suppression-debt convergence in `xiuxian-daochang` by cleaning the next
priority queue at six occurrences.

## Why

The queue-based cleanup sequence remains effective: remove the smallest
remaining groups first to keep risk low and progress measurable.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced panic-style extraction with explicit handling in:
   - `tests/agent/memory/recall_credit.rs`
   - `tests/agent/reflection.rs`
2. Applied invariant-safe branch replacements:
   - `expect`/`expect_err` replaced by explicit `match` with meaningful panic
     paths.
   - lifecycle transition assertions moved to explicit `if let Err(error)` with
     error-carrying panic diagnostics.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- `xiuxian-daochang` tests compile succeeded.
- `xiuxian-daochang` remained pedantic-clean.
- existing workspace sibling warnings remained unchanged
  (`xiuxian-wendao`, `xiuxian-zhixing`, `xiuxian-qianhuan`).
- marker-file count in `xiuxian-daochang/tests` dropped from `21` to `19`.

## Outcome

Convergence moved from the six-occurrence tier to the seven-occurrence tier and
above.

## Next Queue

Prioritize seven-occurrence files:

- `tests/agent_memory_scope_isolation.rs`
- `tests/mcp_discover_cache.rs`
- `tests/mcp_pool_reconnect.rs`
