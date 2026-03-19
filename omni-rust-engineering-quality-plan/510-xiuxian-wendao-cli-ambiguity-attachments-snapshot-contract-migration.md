# 510. Xiuxian Wendao CLI Ambiguity and Attachments Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the Wave 10 migration of the remaining `wendao` CLI `ambiguity` and `attachments` coverage from the internal `test_wendao_cli` wrapper into an explicit external fixture-backed snapshot contract binary.

The migrated files were:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/ambiguity.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/attachments.rs`

They have now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_ambiguity_attachments_contracts.rs`

New fixture roots were added under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/ambiguity/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/attachments/`

## Why This Change Was Needed

After the `search`, `related`, and `agentic` waves, the next cohesive CLI batch was the pair of remaining wrapper families: `ambiguity` and `attachments`.

Keeping them under the internal `test_wendao_cli` wrapper would have preserved the exact indirection the user asked to eliminate. This wave moved both families into a committed fixture-and-snapshot structure and deleted the superseded wrapper tests immediately.

## What Changed

### 1) Added an external `ambiguity` + `attachments` contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_ambiguity_attachments_contracts.rs`

The new binary covers:

- `metadata` ambiguous stem reporting,
- `resolve` ambiguous candidate enumeration,
- `neighbors` ambiguous stem reporting,
- `related` ambiguous stem reporting,
- attachment filtering by `--kind` and `--ext`,
- normalization of `file://` and absolute attachment targets.

### 2) Added dedicated support modules for the new snapshots

Added:

- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_ambiguity_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_attachments_contract_support.rs`

These helpers materialize committed fixture trees and normalize CLI payloads into stable JSON snapshots before comparing them with committed expected fixtures.

### 3) Added committed fixture roots for all six migrated scenarios

Added fixture families under `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/`:

- `ambiguity/metadata_reports_ambiguous_stem_candidates/`
- `ambiguity/resolve_returns_candidates/`
- `ambiguity/neighbors_reports_ambiguous_stem_candidates/`
- `ambiguity/related_reports_ambiguous_stem_candidates/`
- `attachments/search_filters_by_ext_and_kind/`
- `attachments/search_normalizes_file_scheme_targets/`

Each scenario now includes committed `input/` documents and `expected/result.json` snapshots.

### 4) Removed the migrated wrapper tests immediately

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/ambiguity.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/attachments.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/mod.rs`

This left only the `cli_commands` family in the wrapper, which was then handled in Wave 11.

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-cli-ambiguity-attachments-contracts cargo check -p xiuxian-wendao --test test_wendao_cli_ambiguity_attachments_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-cli-wrapper-check cargo check -p xiuxian-wendao --test test_wendao_cli --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-cli-ambiguity-attachments-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_wendao_cli_ambiguity_attachments_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-cli-ambiguity-attachments-contracts cargo clippy -p xiuxian-wendao --test test_wendao_cli_ambiguity_attachments_contracts -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-cli-wrapper-check cargo clippy -p xiuxian-wendao --test test_wendao_cli -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check` completed for the new external binary and the reduced wrapper.
- `cargo nextest run` passed for `test_wendao_cli_ambiguity_attachments_contracts` (`6 passed, 0 skipped`).
- `cargo clippy` completed for the new binary and the reduced wrapper.
- The warnings observed during validation came from pre-existing library code in `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/gateway.rs` and `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/ui.rs`.
