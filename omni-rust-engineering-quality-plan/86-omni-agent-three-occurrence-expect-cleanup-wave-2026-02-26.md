# 修仙道场 (Xiuxian Daochang) Three-Occurrence Expect Cleanup Wave (2026-02-26)

## Scope

Continue `xiuxian-daochang` suppression-debt convergence by cleaning the next
priority batch where each file had three remaining
`expect`/`expect_err`/`unwrap` occurrences.

## Why

The three-occurrence queue is the next highest-yield lane after the
single- and two-occurrence waves. Clearing it keeps convergence predictable
and reduces noise in strict lint signals.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced panic-style extraction with explicit handling in:
   - `tests/agent/graph_planner.rs`
   - `tests/discover_cache_valkey_precedence.rs`
   - `tests/gateway_validation.rs`
   - `tests/jobs_scheduler.rs`
2. Applied consistent root-cause patterns:
   - `expect`/`expect_err` -> explicit `match` with clear panic paths on
     invariant violation.
   - deterministic validation calls -> explicit `if let Err(error)` branches.
   - listener binding/address extraction -> explicit `match` with context-rich
     panic messages.

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
- `xiuxian-daochang` remained clean under pedantic clippy.
- Existing workspace sibling warnings in `xiuxian-wendao` and
  `xiuxian-zhixing` remained unchanged.
- marker file count in `xiuxian-daochang/tests` dropped from `35` to `31`.

## Outcome

The next queue tier is now cleared with no new suppressions and no behavior
loosening. Convergence remains root-cause-first and evidence-backed.

## Next Queue

Prioritize `4`-occurrence files:

- `tests/agent_session_context.rs`
- `tests/channels_discord_ingress.rs`
- `tests/channels_telegram_polling.rs`
- `tests/channels_telegram_slash_authorization.rs`
- `tests/gateway/http/runtime.rs`
- `tests/session_redis.rs`
