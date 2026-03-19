# 修仙道场 (Xiuxian Daochang) Five-Occurrence Expect Cleanup Wave (2026-02-26)

## Scope

Continue suppression-debt convergence in `xiuxian-daochang` by cleaning the
five-occurrence queue for `expect`/`expect_err`/`unwrap`.

## Why

After clearing up through the four-occurrence tier, this queue keeps the same
high-confidence, low-regression cleanup rhythm while preserving strict quality
signals.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced panic-style extraction with explicit handling in:
   - `tests/agent/memory_recall_state.rs`
   - `tests/mcp_pool_hard_timeout.rs`
   - `tests/runtime_agent_factory/inference.rs`
   - `tests/test_support_parsers.rs`
2. Preserved behavior with explicit branch semantics:
   - invariant-required `Option`/`Result` extraction uses `match` + clear panic.
   - expected-error assertions use explicit `match` on success/error paths.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- `xiuxian-daochang` test-target compile succeeded.
- `xiuxian-daochang` remained pedantic-clean.
- workspace sibling warnings in `xiuxian-wendao` and `xiuxian-zhixing`
  remained unchanged.
- marker-file count in `xiuxian-daochang/tests` dropped from `25` to `21`.

## Outcome

The convergence baseline is now in the six-occurrence tier and above.

## Next Queue

Prioritize six-occurrence files:

- `tests/agent/memory/recall_credit.rs`
- `tests/agent/reflection.rs`
