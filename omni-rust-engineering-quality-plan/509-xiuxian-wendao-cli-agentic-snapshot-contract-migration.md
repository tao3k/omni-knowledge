# 509. Xiuxian Wendao CLI Agentic Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the Wave 9 migration of the remaining `wendao` CLI `agentic` family in `xiuxian-wendao` from the internal `test_wendao_cli/agentic` module tree into one explicit external fixture-backed snapshot contract binary.

The migrated internal tree was:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/`

It has now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_agentic_contracts.rs`

New fixture roots were added under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/agentic/`

## Why This Change Was Needed

After the `search` and `related` migration, the next cohesive batch was the whole CLI `agentic` family. Keeping it under the internal `test_wendao_cli` tree would have preserved the exact indirection layer the user asked to remove.

Unlike earlier CLI waves, `agentic` still relied on inline temporary directory setup and direct assertions. This wave converted it into the same explicit snapshot-contract structure used elsewhere in the Wendao suite.

## What Changed

### 1) Added an external `agentic` contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli_agentic_contracts.rs`

The new binary covers:

- suggestion log / recent / decide / decisions flow,
- planning runtime budgets,
- execution runtime budgets and telemetry,
- persisted-suggestion behavior and duplicate suppression,
- discovery-quality signal contracts,
- verbose monitor dashboard output,
- promoted overlay materialization,
- promoted overlay prefix isolation,
- promoted overlay mixed alias resolution,
- provisional overlay isolation before promotion.

### 2) Added dedicated fixture-backed support modules

Added:

- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_agentic_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_agentic_log_flow_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_agentic_planning_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_agentic_execution_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_agentic_overlay_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_agentic_prefix_support.rs`

Also narrowed:

- `packages/rust/crates/xiuxian-wendao/tests/support/wendao_cli_command_contract_support.rs`

That narrowing removed unnecessary runtime coupling from the non-agentic CLI wrapper and reduced warning noise after the migration.

### 3) Added new snapshot fixture roots

Added fixture families under `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/agentic/`:

- `log_flow/`
- `planning/`
- `execution/`
- `overlay/`

Each scenario now includes committed `input/` documents and `expected/result.json` snapshots.

### 4) Removed the migrated internal tree immediately

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/mod.rs`

Why this matters:

- the migrated `agentic` family no longer lives behind the internal wrapper,
- the repository now reflects the user's requirement to delete superseded tests immediately after migration,
- the remaining internal CLI wrapper is reduced to `ambiguity`, `attachments`, `cli_commands`, and shared support only.

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-cli-agentic-contracts cargo check -p xiuxian-wendao --test test_wendao_cli_agentic_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-cli-wrapper-check cargo check -p xiuxian-wendao --test test_wendao_cli --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-cli-agentic-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_wendao_cli_agentic_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-cli-agentic-contracts cargo clippy -p xiuxian-wendao --test test_wendao_cli_agentic_contracts -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-cli-wrapper-check cargo clippy -p xiuxian-wendao --test test_wendao_cli -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check` completed cleanly for the new `agentic` binary and the reduced wrapper.
- `cargo nextest run` passed for `test_wendao_cli_agentic_contracts` (`10 passed, 0 skipped`).
- `cargo clippy` completed for both targeted test binaries.
- The only warnings observed during validation were unrelated pre-existing library warnings in `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/gateway.rs`.
