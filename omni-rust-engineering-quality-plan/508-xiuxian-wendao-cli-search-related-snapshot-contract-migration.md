# 508. Xiuxian Wendao CLI Search and Related Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the Wave 8 migration of the `wendao` CLI retrieval surface in `xiuxian-wendao` from the internal `test_wendao_cli` module tree into explicit external fixture-backed snapshot contract binaries.

The migrated internal trees were:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/`

They have now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_search_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_related_contracts.rs`

Supporting snapshot helpers were moved into `packages/rust/crates/xiuxian-wendao/tests/support/` with contract-oriented names.

## Why This Change Was Needed

After the `LinkGraph` migration, the next cohesive batch was the user-visible CLI retrieval surface. Both `search` and `related` were already fixture-contract oriented, but they were still nested under the internal `test_wendao_cli` module tree.

Keeping migrated suites inside that tree would have preserved the exact duplication the user asked to remove. Externalizing them makes the contract boundary explicit and lets the internal wrapper shrink to only the non-migrated CLI families.

## What Changed

### 1) Added external `search` and `related` contract binaries

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_search_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_related_contracts.rs`

The new `search` binary covers:

- basic retrieval output and sorting behavior,
- query directives and legacy sort rejection,
- semantic and temporal filter flags,
- link-filter and related-PPR search flags,
- provisional overlay injection and engine-default overlay behavior.

The new `related` binary covers:

- PPR parameter handling,
- verbose diagnostics and monitor payloads.

### 2) Moved fixture contract support to `tests/support/`

Added or relocated:

- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_search_basic_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_search_directives_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_search_link_filters_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_search_provisional_overlay_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_related_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_contract_runtime_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_command_contract_support.rs`

This keeps the external binaries self-contained without disturbing the remaining internal CLI suites.

### 3) Removed the migrated internal CLI trees immediately

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli.rs`

Why this matters:

- the migrated command families no longer live behind an internal indirection layer,
- the repository now reflects the user's requirement to delete superseded tests immediately after migration,
- the remaining internal CLI wrapper is smaller and only hosts non-migrated suites.

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-cli-search-contracts cargo check -p xiuxian-wendao --test test_wendao_cli_search_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-cli-related-contracts cargo check -p xiuxian-wendao --test test_wendao_cli_related_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-cli-wrapper-check cargo check -p xiuxian-wendao --test test_wendao_cli --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-cli-search-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_wendao_cli_search_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-cli-related-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_wendao_cli_related_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-cli-search-contracts cargo clippy -p xiuxian-wendao --test test_wendao_cli_search_contracts -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-cli-related-contracts cargo clippy -p xiuxian-wendao --test test_wendao_cli_related_contracts -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-cli-wrapper-check cargo clippy -p xiuxian-wendao --test test_wendao_cli -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check` completed cleanly for the two new binaries and the remaining wrapper.
- `cargo nextest run` passed for `test_wendao_cli_search_contracts` (`13 passed`).
- `cargo nextest run` passed for `test_wendao_cli_related_contracts` (`2 passed`).
- `cargo clippy` completed cleanly for all targeted test binaries.
- No expected snapshot updates were required in this wave.
