# 修仙道场 (Xiuxian Daochang) Missing-Docs Reduction Wave: Contracts + Settings + Agent State (2026-02-27)

## Scope

After completing marker-zero convergence in `xiuxian-daochang/tests`, continue
source-quality convergence by reducing `missing_docs` warnings in high-density
production modules.

## Implemented Changes

1. Added comprehensive API docs for runtime settings contracts:
   - `packages/rust/crates/xiuxian-daochang/src/config/settings/types.rs`
2. Added field/variant/function docs for route/plan/governance contracts:
   - `packages/rust/crates/xiuxian-daochang/src/contracts/route_trace.rs`
   - `packages/rust/crates/xiuxian-daochang/src/contracts/graph_plan.rs`
   - `packages/rust/crates/xiuxian-daochang/src/contracts/omega.rs`
   - `packages/rust/crates/xiuxian-daochang/src/contracts/discover.rs`
   - `packages/rust/crates/xiuxian-daochang/src/contracts/memory_gate.rs`
3. Added docs for observability snapshots and inspection APIs:
   - `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_metrics.rs`
   - `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_state/types.rs`
   - `packages/rust/crates/xiuxian-daochang/src/agent/context_budget_state.rs`
4. Added docs for additional public API surfaces:
   - `packages/rust/crates/xiuxian-daochang/src/jobs/manager/types.rs`
   - `packages/rust/crates/xiuxian-daochang/src/gateway/http/types.rs`
   - `packages/rust/crates/xiuxian-daochang/src/embedding/client/init.rs`
5. Kept suppression policy strict:
   - no new `#[allow(missing_docs)]` introduced.

## Validation Notes

Executed during this wave:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
```

Environment blocker observed:

- Both clippy commands can fail before lint completion because
  `mistralrs-quant`/`mistralrs-paged-attn` build scripts require macOS Metal
  toolchain binary `metal`, which is currently unavailable in this environment.
- Example error: `cannot execute tool 'metal' due to missing Metal Toolchain`.

Local marker verification remains clean:

```bash
rg -o "clippy::too_many_lines|clippy::too_many_arguments" \
  packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' \
  | sed 's/.*://g' | sort | uniq -c | sort -nr
```

Result: empty output (zero occurrences).

## Outcome

This wave significantly reduced `missing_docs` debt in core contract and
settings surfaces and preserved marker-zero status for
`too_many_lines`/`too_many_arguments` in `xiuxian-daochang/tests`, while documenting
that full clippy verification is currently gated by host toolchain
availability (`metal`).
