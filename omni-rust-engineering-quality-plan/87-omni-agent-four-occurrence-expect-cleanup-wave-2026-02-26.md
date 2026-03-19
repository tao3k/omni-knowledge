# 修仙道场 (Xiuxian Daochang) Four-Occurrence Expect Cleanup Wave (2026-02-26)

## Scope

Continue `xiuxian-daochang` test-lane convergence by cleaning the next queue where
files had four remaining `expect`/`expect_err`/`unwrap` occurrences.

## Why

After clearing one-through-three occurrence queues, this wave keeps the
reduction slope stable while preserving test behavior and strict lint signals.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced panic-style extraction with explicit handling in:
   - `tests/agent_session_context.rs`
   - `tests/channels_discord_ingress.rs`
   - `tests/channels_telegram_polling.rs`
   - `tests/channels_telegram_slash_authorization.rs`
   - `tests/gateway/http/runtime.rs`
   - `tests/session_redis.rs`
2. Fixed one additional source-level clippy blocker discovered during
   validation:
   - `src/agent/bootstrap/tests.rs`
   - replaced remaining `expect()` in temp directory preparation with explicit
     error branch.
3. Resolved follow-up pedantic warnings introduced during replacement:
   - `tests/gateway/http/runtime.rs`
   - replaced `Err(_)` wildcard arms with explicit error-bound branches.

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
- `xiuxian-daochang` remained pedantic-clean after removing clippy blocker and
  warning regressions.
- existing workspace sibling warnings remained only in `xiuxian-wendao` and
  `xiuxian-zhixing`, unchanged by this wave.
- marker file count in `xiuxian-daochang/tests` dropped from `31` to `25`.

## Outcome

The convergence lane remains stable and root-cause-first. The next remaining
queue starts at five-occurrence files.

## Next Queue

Prioritize five-occurrence files:

- `tests/agent/memory_recall_state.rs`
- `tests/mcp_pool_hard_timeout.rs`
- `tests/runtime_agent_factory/inference.rs`
- `tests/test_support_parsers.rs`
