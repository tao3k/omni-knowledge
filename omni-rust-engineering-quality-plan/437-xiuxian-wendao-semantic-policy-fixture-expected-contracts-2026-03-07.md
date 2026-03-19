# 437. Xiuxian Wendao Semantic-Policy Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the Wendao LinkGraph test-architecture slice that migrates
`semantic_policy` from direct assertions to expected JSON contracts under the
fixture-first layout.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy.rs`
was small, but it still mixed transient corpus setup and direct assertions.
That shape broke the consistency of the broader fixture-first migration already
applied across `test_link_graph`.

## What Changed

### 1) Added a semantic-policy fixture support module

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy_fixture_support.rs`

This module owns:

- semantic-policy scenario materialization,
- query-parse JSON projection,
- planned-payload semantic-policy projection,
- explicit `summary_only` / `all` label normalization.

### 2) Migrated the semantic-policy suite to expected JSON contracts

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy.rs`

New scenarios:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/semantic_policy/parse_directives/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/semantic_policy/planned_payload/...`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/semantic_policy/parse_directives/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/semantic_policy/planned_payload/expected/result.json`

What the contracts now cover:

- semantic-policy directive parsing from query text,
- semantic-policy preservation in planned payload options,
- semantic-policy propagation into retrieval-plan payloads.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/semantic_policy/parse_directives/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/semantic_policy/planned_payload/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/semantic_policy/planned_payload/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/semantic_policy/planned_payload/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_link_graph --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_policy
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_policy`
  passed (`3 passed, 81 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines`
  completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/436-xiuxian-wendao-refresh-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/437-xiuxian-wendao-semantic-policy-fixture-expected-contracts-2026-03-07.md`
