# 516. Xiuxian Wendao Final Enhancer and HMAS Snapshot Migration

Date: 2026-03-08

## Scope

This shard records the final snapshot-wrapper retirement wave for `xiuxian-wendao`, migrating the last two snapshot-worthy wrapper families into one external contract binary.

The new external binary is:

- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer_hmas_contracts.rs`

The new support helpers are:

- `packages/rust/crates/xiuxian-wendao/tests/support/enhancer_snapshot_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/hmas_snapshot_contract_support.rs`

The retired wrappers are:

- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_hmas.rs`

The retired wrapper trees are:

- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/`
- `packages/rust/crates/xiuxian-wendao/tests/test_hmas/`

The committed fixture roots are:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/enhancer/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/hmas/`

## What Changed

- Replaced the `test_enhancer` and `test_hmas` wrapper families with one fixture-backed contract binary.
- Moved enhancer expectations into stable JSON snapshots for:
  - frontmatter parsing,
  - markdown config block extraction,
  - markdown config link normalization,
  - note enhancement output,
  - inferred relation output.
- Moved HMAS validation expectations into stable JSON snapshots for:
  - valid blackboard reports,
  - missing digital-thread detection,
  - invalid JSON payload detection,
  - digital-thread source/constraint/confidence validation.

## Normalization Strategy

The final wave keeps only stable, contract-level fields in snapshots:

- Enhancer config-link snapshots sort top-level ids to avoid `HashMap` iteration nondeterminism.
- Enhancer config-block snapshots trim trailing newline noise from fenced block payloads.
- Enhancer note-enhancement snapshots preserve the real `ref_stats.by_type` shape, including `"none"` buckets for untyped refs.
- HMAS validation snapshots preserve stable counts, codes, kinds, and deterministic line numbers.
- HMAS `invalid_json_payload` messages are normalized to `<invalid-json-payload>` so serde wording changes do not create churn.

## Validation

Executed and passed:

- `CARGO_TARGET_DIR=/tmp/xiuxian-enhancer-hmas-contracts cargo check -p xiuxian-wendao --test test_enhancer_hmas_contracts --message-format short`
- `CARGO_TARGET_DIR=/tmp/xiuxian-enhancer-hmas-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_enhancer_hmas_contracts`
- `CARGO_TARGET_DIR=/tmp/xiuxian-enhancer-hmas-contracts cargo clippy -p xiuxian-wendao --test test_enhancer_hmas_contracts -- -W clippy::too_many_lines`

Observed non-blocking warning noise remained limited to pre-existing runtime-config dead-code warnings in:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/gateway.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/ui.rs`

## Final Audit Closure

The snapshot-wrapper audit is now closed.

Remaining wrapper+directory families should stay modular for one of three reasons:

- pure unit/data-model coverage,
- stateful integration behavior that is clearer with explicit assertions,
- benchmark/performance suites where snapshots add no value.

There are no additional high-value snapshot-wrapper migration candidates left in `packages/rust/crates/xiuxian-wendao/tests/` after this wave.

## Result

`xiuxian-wendao` no longer has outstanding wrapper families that should be migrated to snapshot contracts. The committed enhancer and HMAS fixture trees are now the final source of truth for those surfaces.
