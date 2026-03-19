# 436. Xiuxian Wendao Refresh Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the next Wendao LinkGraph test-architecture slice: migrating
`refresh` from inline tempdir orchestration and direct assertions to
fixture-backed `input/expected` contracts.

This slice keeps the same fixture-first rules established in the recent Wendao
migrations:

- corpus inputs live under `tests/fixtures/.../input/`,
- expected outcomes live under `tests/fixtures/.../expected/`,
- scenario projection logic stays in a focused domain support module,
- no separate snapshot root is introduced.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs` was
small, but it still mixed too many responsibilities inside the test body:

- initial corpus construction,
- incremental update orchestration,
- deletion orchestration,
- refresh-mode assertions,
- search-result assertions,
- stats assertions.

That shape is workable for two tests, but it does not scale and it does not fit
the fixture-first standard now used by the rest of the LinkGraph test lane.

## What Changed

### 1) Added a dedicated refresh fixture support module

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh_fixture_support.rs`

This module owns:

- refresh scenario materialization,
- stable JSON projection for refresh hits,
- stable JSON projection for refresh stats,
- refresh-mode normalization (`noop` / `delta` / `full`),
- sequence-style contract shaping for update/delete flows.

Why this matters:

- refresh tests now read as behavior-only orchestration,
- all contract shaping lives in one explicit place,
- additional refresh scenarios can extend the same surface without rebuilding
  helper code.

### 2) Migrated the full `refresh` suite to scenario fixtures

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`

New scenario fixture trees:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/incremental_update_and_delete/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/threshold_modes/...`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/incremental_update_and_delete/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/threshold_modes/expected/result.json`

What the contracts now cover:

- old-hit visibility before incremental update,
- `delta` refresh after content mutation,
- updated-hit visibility after mutation,
- `delta` refresh after deletion,
- final graph stats after deletion,
- `noop` refresh for empty change sets,
- `full` refresh when threshold forcing is triggered,
- final hits and stats after a forced full rebuild.

Why this matters:

- refresh behavior is now reviewable as serialized state transitions,
- scenario input and expected output live in one place,
- regression auditing is easier when refresh semantics change.

### 3) Extended the shared LinkGraph test root explicitly

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

What changed:

- imported the new refresh support surface,
- registered the support module alongside the other fixture-based test slices.

Why this matters:

- the `test_link_graph` root remains explicit and modular,
- the fixture-first pattern stays consistent across the entire lane.

## Architectural Takeaways

### Refresh is a state-transition contract

The important thing about refresh tests is not just that one search still works.
The important thing is the sequence of state transitions: before mutation,
after mutation, after deletion, after threshold escalation. Fixture contracts are
well suited to this shape.

### Small suites should still follow the same architecture rule

A test file being short is not a reason to keep mixed concerns inline. The
standard should remain domain fixtures plus dedicated projection support.

### Test-lane consistency matters more than local convenience

Migrating `refresh` keeps the LinkGraph lane architecturally coherent. That is
more valuable than preserving a locally convenient older style in one leftover
module.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/incremental_update_and_delete/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/incremental_update_and_delete/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/incremental_update_and_delete/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/threshold_modes/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/threshold_modes/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/threshold_modes/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph refresh
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph refresh`
  passed (`3 passed, 81 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines`
  completed cleanly.

## Limits and Next Slice

The recent fixture-first migration wave has now covered the largest remaining
assertion-heavy slices in `test_link_graph`.

The next follow-up should be a smaller audit pass:

- check whether any remaining `test_link_graph` modules still hide inline
  corpus construction,
- remove any now-redundant test-root imports or support wiring,
- keep new scenarios aligned to the `input/expected` layout instead of letting
  ad hoc inline setup grow back.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/435-xiuxian-wendao-markdown-attachments-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/436-xiuxian-wendao-refresh-fixture-expected-contracts-2026-03-07.md`
