# 449. Xiuxian Wendao PPR and CLI Related Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the migration of one remaining weighted-seed PPR regression
lane and the `wendao related` CLI lanes onto fixture-backed contracts.

## Why This Change Was Needed

After the previous link-graph inline-corpus migration, two related debts still
remained in `xiuxian-wendao/tests`:

- `test_ppr_weight_precision.rs` still built its corpus inline with
  `tempdir()` and used `#[tokio::test]` despite having no async work.
- `test_wendao_cli/related/*` still built notebooks inline and relied on
  imperative assertion helpers instead of stable fixture contracts.

The result was that both the graph core and the CLI surface were still mixing
runtime corpus assembly with hand-written assertion logic.

## What Changed

### 1) Migrated `test_ppr_weight_precision.rs` onto fixture-backed input and expected output

The test now reuses the existing weighted-seed graph input fixture under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/ppr_weighting/non_uniform_seed_bias/input/`

and asserts against a dedicated expected contract under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/ppr_precision/weighted_seed_precision_impact/expected/result.json`

It was also converted from `#[tokio::test]` to plain `#[test]`.

### 2) Added a domain-specific fixture support module for CLI `related` contracts

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/fixture_contract_support.rs`

This support file materializes CLI input fixtures and projects runtime JSON into
stable contract shapes for:

- plain `related` result rows,
- verbose `related` payloads,
- PPR diagnostics invariants,
- monitor phase labels,
- promoted-overlay metadata.

### 3) Migrated the `wendao related` CLI lanes onto one shared fixture scenario

The two CLI tests now reuse a single scenario root:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/related/linear_chain/input/`

with separate expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/related/linear_chain/expected/related_with_ppr_flags.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/related/linear_chain/expected/related_verbose_with_diagnostics.json`

This removes duplicated notebook setup while keeping the plain and verbose CLI
surfaces independently documented.

### 4) Removed the obsolete imperative assertion helpers from the CLI related lane

Deleted:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/diagnostics_assertions.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/monitor_assertions.rs`

The lane now has one contract-oriented support file instead of two small
assertion-only helper files.

## Architectural Takeaways

- Reusing an existing input fixture with a new expected contract is preferable
  to cloning tiny corpora just to satisfy a different regression lane.
- CLI tests benefit from the same `input/expected` discipline as library tests;
  the command surface should be documented as a contract, not only as ad hoc
  asserts.
- When verbose payloads include unstable telemetry, project them into semantic
  invariants instead of snapshotting raw durations.
- If a lane fully moves to fixture contracts, remove the old assertion helpers
  immediately so the structure stays singular.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_ppr_weight_precision.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/related_command_accepts_ppr_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/related_verbose_includes_diagnostics.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/ppr_precision/weighted_seed_precision_impact/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/related/linear_chain/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/related/linear_chain/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/related/linear_chain/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/related/linear_chain/input/docs/d.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/related/linear_chain/expected/related_with_ppr_flags.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/related/linear_chain/expected/related_verbose_with_diagnostics.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_ppr_weight_precision --test test_wendao_cli --message-format short
cargo nextest run -p xiuxian-wendao --test test_ppr_weight_precision --test test_wendao_cli
cargo clippy -p xiuxian-wendao --test test_ppr_weight_precision --test test_wendao_cli -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ...` completed cleanly.
- `cargo nextest run ...` passed (`37 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly after splitting the verbose CLI contract
  projection into smaller helpers.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/448-xiuxian-wendao-link-graph-inline-corpus-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/449-xiuxian-wendao-ppr-and-cli-related-fixture-contracts-2026-03-07.md`
