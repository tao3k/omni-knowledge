# 201. 修仙道场 (Xiuxian Daochang) Large-Futures Convergence and Clippy Unblock Wave (2026-02-28)

## Scope

- Primary crate: `packages/rust/crates/xiuxian-daochang`
- Supporting unblock crates:
  - `packages/rust/crates/xiuxian-skills`
  - `packages/rust/crates/xiuxian-zhixing`

## Why This Wave

After regression-closure wave 200, strict clippy still reported a large set of
`clippy::large_futures` warnings centered on `Agent::run_turn` call sites. In
addition, strict clippy for `xiuxian-daochang` was intermittently blocked by upstream
compile/lint issues in dependency crates in the same workspace graph.

## Changes Implemented

1. Reduced `run_turn` future size at the source.
   - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/mod.rs`
   - Wrapped `run_react_loop(...)` with `Box::pin(...)` inside `Agent::run_turn`,
     shrinking the propagated future footprint at callers.

2. Removed cast-truncation warning in zhenfa unit tests.
   - `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/tests.rs`
   - Replaced `f64 as f32` cast with `ToPrimitive::to_f32()`.

3. Fixed workspace compile blocker in scanner metadata parse path.
   - `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/parse/metadata.rs`
   - Replaced move-prone fallback and refactored into `map_or_else(...)` to avoid
     ownership error and pedantic warning.

4. Cleared remaining pedantic warnings in zhixing indexer support path.
   - `packages/rust/crates/xiuxian-zhixing/src/wendao/indexer/resource_graph.rs`
   - Applied clippy suggestions for nested-or pattern and `map_or_else(...)`.

No lint suppressions were introduced.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-skills -p xiuxian-zhixing -p xiuxian-daochang
```

- Result: pass

2. Strict clippy (required command for touched `xiuxian-daochang` crate):

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-daochang cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass
- Output: no clippy warnings emitted in final run for checked crates in this path.

3. Targeted runtime regression set (run-turn heavy lanes):

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang \
  --test agent_injection \
  --test agent_context_window_recovery \
  --test zhenfa_tool_bridge \
  --test agent_suite \
  --status-level fail --failure-output immediate-final --no-fail-fast
```

- Result: pass
- Summary: `21 passed`, `0 failed`, `1 skipped`

4. Full crate suite:

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang
```

- Result: pass
- Summary: `653 passed`, `0 failed`, `30 skipped`

## Outcome

- `xiuxian-daochang` strict clippy is now unblocked and clean under this wave's
  execution path.
- `run_turn`-fanout `large_futures` warning pressure has been structurally reduced
  via source-side future boxing rather than per-call-site suppressions.
- Workspace dependency blockers encountered during the wave were fixed without
  introducing temporary bypasses.
