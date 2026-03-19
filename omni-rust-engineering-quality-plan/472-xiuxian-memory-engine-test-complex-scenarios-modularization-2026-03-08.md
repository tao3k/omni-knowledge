# 472. Xiuxian Memory Engine `test_complex_scenarios` Modularization

Date: 2026-03-08

## Scope

This shard records the structural decomposition of
`packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios.rs`.

## Why This Change Was Needed

The integration test entrypoint had grown into a 685-line mixed-responsibility
file that combined reinforcement-learning, retrieval, decay, persistence,
performance, and incremental-learning scenarios in one place.

That made the suite harder to evolve and violated the repository rule that test
code should be split by concern rather than accumulated into large single files.

## What Changed

Reduced the root integration-test entrypoint to a thin launcher:

- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios.rs`

Created a dedicated directory module:

- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/mod.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/support.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/reinforcement.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/retrieval.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/decay.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/persistence.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/performance.rs`

Responsibility split:

- `support.rs`: shared `TestResult` alias and store builders;
- `reinforcement.rs`: feedback adaptation, convergence, and incremental-learning scenarios;
- `retrieval.rs`: two-phase retrieval, multi-hop reasoning, conflicting experiences, and utility/similarity trade-off;
- `decay.rs`: Q-value decay scenario;
- `persistence.rs`: save/load recovery scenario;
- `performance.rs`: batch storage and recall timing scenario.

## Architectural Takeaways

- Scenario-heavy integration suites should still be decomposed by behavioral
  slice; narrative test intent does not justify a monolithic file.
- A small support module can remove repeated store-setup noise while keeping the
  assertions local to the scenario files.
- Directory modules provide a stable growth path for future scenario additions
  without recreating a single-file bottleneck.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-memory-engine --tests
cargo nextest run -p xiuxian-memory-engine --test test_complex_scenarios --no-fail-fast
cargo clippy -p xiuxian-memory-engine -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-memory-engine --tests` passed.
- `cargo nextest run -p xiuxian-memory-engine --test test_complex_scenarios --no-fail-fast`
  passed (`10 passed, 0 skipped`).
- `cargo clippy -p xiuxian-memory-engine -- -W clippy::too_many_lines` passed.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/mod.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/support.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/reinforcement.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/retrieval.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/decay.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/persistence.rs`
- `packages/rust/crates/xiuxian-memory-engine/tests/test_complex_scenarios/performance.rs`
