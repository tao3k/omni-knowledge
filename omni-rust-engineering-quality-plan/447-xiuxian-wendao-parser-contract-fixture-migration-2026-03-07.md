# 447. Xiuxian Wendao Parser Contract Fixture Migration

Date: 2026-03-07

## Scope

This shard records the migration of the Wendao parser contract lane from
snapshot assertions to fixture-backed expected contracts.

## Why This Change Was Needed

The parser lane was the last remaining consumer of
`tests/support/snapshot_assertions.rs` inside `xiuxian-wendao/tests`.

As long as it remained on the old structure, the crate still carried two
parallel contract systems:

- fixture-backed JSON contracts for the modernized lanes,
- snapshot-backed JSON contracts for parser coverage.

## What Changed

### 1) Renamed the parser lane to `test_parser_contracts.rs`

Replaced:

- `packages/rust/crates/xiuxian-wendao/tests/test_parser_snapshots.rs`
- `packages/rust/crates/xiuxian-wendao/tests/parser_snapshots/`

With:

- `packages/rust/crates/xiuxian-wendao/tests/test_parser_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/`

### 2) Moved markdown expected outputs under fixtures

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/skill_registry/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/reference_relations/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/frontmatter/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/config_blocks/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/link_targets/expected/result.json`

### 3) Moved the ORG placeholder slot under fixtures

New placeholder directory:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/org/README.md`

The reserved ORG slot still exists, but it now follows the same fixture-root
convention as every other parser contract.

### 4) Removed the legacy snapshot infrastructure from this crate

Deleted:

- `packages/rust/crates/xiuxian-wendao/tests/snapshots/parser/...`
- `packages/rust/crates/xiuxian-wendao/tests/support/snapshot_assertions.rs`

After this migration, `xiuxian-wendao/tests` no longer depends on
`snapshot_assertions`.

## Architectural Takeaways

- Once the last snapshot consumer in a crate is migrated, remove the shared
  snapshot helper immediately; dead infrastructure invites regression.
- Parser contracts benefit from the same per-scenario `expected/result.json`
  layout as runtime and registry tests, even when the inputs are embedded strings
  instead of fixture trees.
- Placeholder coverage for future features should live in the same structural
  convention as active tests so the expansion path stays obvious.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_parser_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/markdown.rs`
- `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/org.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/skill_registry/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/reference_relations/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/frontmatter/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/config_blocks/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/markdown/link_targets/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/parser/org/README.md`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_parser_contracts --message-format short
cargo nextest run -p xiuxian-wendao --test test_parser_contracts
cargo clippy -p xiuxian-wendao --test test_parser_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ...` completed cleanly.
- `cargo nextest run ...` passed (`6 passed, 1 skipped`).
- `cargo clippy ...` completed cleanly.
- `rg -n "snapshot_assertions|assert_snapshot_eq\(" packages/rust/crates/xiuxian-wendao/tests -g '!**/fixtures/**'` returned no matches.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/446-xiuxian-wendao-repository-internal-manifest-contract-fixture-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/447-xiuxian-wendao-parser-contract-fixture-migration-2026-03-07.md`
