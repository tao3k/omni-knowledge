# 511. Xiuxian Wendao CLI Commands Wrapper Retirement

Date: 2026-03-08

## Scope

This shard records the Wave 11 migration of the final `wendao` CLI `cli_commands` family from the internal `test_wendao_cli` wrapper into an explicit external fixture-backed snapshot contract binary, followed by retirement of the wrapper itself.

The migrated wrapper tree was:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/`

It has now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_commands_contracts.rs`

New fixture roots were added under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/cli_commands/`

## Why This Change Was Needed

After Wave 10, the only remaining wrapper-based CLI tests were five `cli_commands` cases. Keeping a dedicated wrapper binary and nested module tree for only those cases would have preserved an obsolete test indirection layer.

This wave finished the migration and removed the wrapper entirely so that the `xiuxian-wendao` CLI coverage now lives only in external snapshot contract binaries.

## What Changed

### 1) Added the final external CLI commands contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_commands_contracts.rs`

The new binary covers:

- trailing `--root` parsing after a subcommand,
- `hmas validate` contract output,
- hierarchical `page-index` output,
- ambiguous `page-index` alias reporting,
- `stats` note and graph counts.

### 2) Added dedicated snapshot support for the remaining command family

Added:

- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_commands_snapshot_contract_support.rs`

This helper materializes committed fixture trees and normalizes command-specific payloads, including the recursive `page-index` tree shape.

### 3) Added committed fixture roots for all five final scenarios

Added fixture families under `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/cli_commands/`:

- `allows_global_root_after_subcommand/`
- `hmas_validate_command/`
- `page_index_emits_hierarchical_roots/`
- `page_index_reports_ambiguous_aliases/`
- `stats_reports_note_counts/`

Each scenario now includes committed `input/` documents and `expected/result.json` snapshots.

### 4) Retired the internal CLI wrapper completely

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/`

This is the terminal state the user asked for: migrated tests now live in external snapshot binaries, and superseded wrapper tests are deleted immediately.

### 5) Applied one narrow compile-enabling fix in runtime config visibility

Updated:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/plan/payload/routing.rs`

Why this was necessary:

- the final target exposed an existing crate-private visibility bug where `routing.rs` referenced `runtime_config::models` through a private module boundary,
- making `runtime_config::models` crate-private and keeping the import explicit was the smallest non-behavioral fix that allowed the new final binary to compile,
- two fresh clippy warnings in `routing.rs` were then cleaned up immediately so the migration would not leave new lint debt.

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-cli-commands-contracts cargo check -p xiuxian-wendao --test test_wendao_cli_commands_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-cli-commands-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_wendao_cli_commands_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-cli-commands-contracts cargo clippy -p xiuxian-wendao --test test_wendao_cli_commands_contracts -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-cli-search-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_wendao_cli_search_contracts
```

Observed outcomes:

- `cargo check` completed for `test_wendao_cli_commands_contracts`.
- `cargo nextest run` passed for `test_wendao_cli_commands_contracts` (`5 passed, 0 skipped`).
- `cargo clippy` completed for the final contract binary.
- A focused regression run of `test_wendao_cli_search_contracts` also passed after the routing cleanup (`13 passed, 0 skipped`).
- Remaining warnings observed during validation came from pre-existing library code in `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/gateway.rs` and `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/ui.rs`.
