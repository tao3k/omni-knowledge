# Xiuxian-Qianji Test Marker Zero Convergence (2026-02-26)

## Scope

Eliminate remaining file-level `clippy::expect_used|clippy::unwrap_used`
markers in `xiuxian-qianji/tests` while preserving pedantic compliance.

## Implemented Changes

1. Removed `clippy::expect_used`/`clippy::unwrap_used` from:
   - `tests/test_qianji_trinity_integration.rs`
   - `tests/test_qianji_precision_research.rs`
   - `tests/test_qianji_master_research.rs`
   - `tests/test_probabilistic_routing.rs`
   - `tests/unit_qianji_execution.rs`
   - `tests/test_qianji_yaml_orchestration.rs`
   - `tests/unit_qianji_safety.rs`
   - `tests/unit_adversarial_loop.rs`
2. Replaced panic-prone extraction paths with explicit `Result` propagation:
   - converted async tests to `Result<(), Box<dyn std::error::Error>>` where
     needed.
   - replaced `.expect(...)` / `.unwrap(...)` with `?` and explicit
     `ok_or_else(...)` checks.
   - replaced `unwrap_err()` branch check with `let Err(error) = ... else`.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-qianji
cargo clippy -p xiuxian-qianji --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-qianji --all-targets -- -W clippy::pedantic
cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines
rg -l "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-qianji/tests | wc -l
```

Result:

- `xiuxian-qianji` tests and all targets pass pedantic checks.
- `xiuxian-qianji` passes `too_many_lines` policy verification.
- marker-file count in `xiuxian-qianji/tests` dropped from `8` to `0`.

## Outcome

`xiuxian-qianji/tests` is now at zero file-level
`clippy::expect_used|clippy::unwrap_used` markers for the current baseline.
