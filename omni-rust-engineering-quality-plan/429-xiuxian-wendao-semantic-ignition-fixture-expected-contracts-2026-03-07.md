# 429. Xiuxian Wendao Semantic-Ignition Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the second Wendao LinkGraph test-architecture migration wave
that extends the fixture-first contract style into the semantic-ignition lane.

The goal of this slice was to keep pushing the same testing standard forward:

- reuse a shared input corpus from `tests/fixtures/.../input/`,
- store expected behavior contracts under `tests/fixtures/.../expected/`,
- remove repeated inline corpus construction from the test suite,
- keep the touched `test_link_graph` lane strict-clippy clean.

## Why This Change Was Needed

After the earlier `quantum_fusion` and `quantum_anchor_batch` migration, the
neighboring `semantic_ignition.rs` suite still repeated the same hybrid corpus
setup inline:

- alpha/beta/gamma Markdown graph creation,
- plain-note fallback creation,
- repeated `TempDir` and `LinkGraphIndex::build(...)` boilerplate,
- assertion-heavy success-path checks instead of stable output contracts.

That left the LinkGraph hybrid-retrieval lane in a mixed testing style.

## What Changed

### 1) Reused the shared hybrid fixture instead of rebuilding the corpus inline

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`

The suite now uses:

- `build_hybrid_fixture()`
- `alpha_leaf_anchor_id()`

instead of writing ad hoc Markdown files for each test.

Why this matters:

- the corpus is now canonical across hybrid-retrieval tests,
- fixture setup noise disappeared from semantic-ignition coverage,
- behavior reads directly from the test body again.

### 2) Converted semantic-ignition outputs into expected JSON contracts

New expected fixtures:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/delegates_and_recovers_trace.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/skips_empty_requests.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/respects_min_vector_score.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/backend_error.json`

The suite now snapshots:

- backend-call count,
- full projected `QuantumContext` output,
- empty-request short-circuit behavior,
- backend-error contract shape.

Why this matters:

- the semantic-ignition lane now matches the fixture-expected style introduced
  in shard `428`,
- regression review is easier because the output contract is visible in one JSON
  document instead of spread across many assertions,
- success and failure contracts now share one consistent storage model.

### 3) Kept projection logic centralized instead of duplicating serialization noise

The migrated suite reuses the existing LinkGraph quantum projection helper:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fixture_support.rs`

Why this matters:

- semantic-ignition tests do not invent their own output formatting rules,
- `QuantumContext` snapshots stay normalized the same way across suites,
- future hybrid tests can keep building on one stable projection contract.

## Architectural Takeaways

### Once a fixture corpus becomes canonical, neighboring suites should converge quickly

Leaving one adjacent suite on manual tempdir setup after a shared fixture exists
creates unnecessary divergence. After the shared hybrid corpus was introduced,
semantic-ignition became the obvious next migration target.

### Snapshot contracts should include control signals, not only domain payloads

For semantic ignition, the important contract is not just the retrieved context.
It also includes whether the backend was called. That is why the expected JSON
stores both `calls` and the projected result.

### Error contracts benefit from the same expected-fixture storage model

Failure-path verification does not need to stay assertion-only by default.
Stable error shape can also be stored as an expected contract when it improves
readability and keeps the suite uniform.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/delegates_and_recovers_trace.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/skips_empty_requests.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/respects_min_vector_score.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/backend_error.json`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition` passed (`4 passed, 80 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines` completed cleanly.

## Limits and Next Slice

This slice keeps the migration tightly focused on the semantic-ignition lane.
Other high-repetition LinkGraph suites still remain, especially the larger
search-oriented modules with many inline corpus builders.

The next migration pass should continue using the same selection rule:

- heavy repeated `write_file(...)` setup,
- stable JSON-shape outputs,
- strong payoff from canonical input and expected contracts.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/428-xiuxian-wendao-link-graph-fixture-expected-snapshot-migration-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/429-xiuxian-wendao-semantic-ignition-fixture-expected-contracts-2026-03-07.md`
