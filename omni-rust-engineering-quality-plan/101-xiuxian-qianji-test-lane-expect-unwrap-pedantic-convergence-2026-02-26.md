# Xiuxian-Qianji Test-Lane Expect/Unwrap Pedantic Convergence (2026-02-26)

## Scope

Continue the second-pass Rust quality sweep by removing panic-based extraction
paths from `xiuxian-qianji` test lanes and converging pedantic checks without
suppression-first changes.

## Implemented Changes

1. Replaced `unwrap` in internal executor tests:
   - `src/executors/formal_audit.rs`
   - converted async tests to `Result<(), String>` and propagated execution
     errors via `?`.
2. Replaced `expect/unwrap` in integration tests:
   - `tests/test_formal_adversarial_audit.rs`
   - `tests/test_scheduler_checkpoint.rs`
   - `tests/test_smart_commit_integration.rs`
   - `tests/test_schema_contracts.rs`
   - converted tests to `Result`-returning style and replaced optional/value
     extraction with explicit `ok_or_else(...)` checks where needed.
3. Cleared remaining pedantic warnings in runtime config tests:
   - `tests/runtime_config.rs`
   - changed helper signatures from pass-by-value to borrowed env references.
   - replaced two `match`-extraction blocks with `let Err(err) = ... else`.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-qianji
cargo clippy -p xiuxian-qianji --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-qianji --all-targets -- -W clippy::pedantic
cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines
```

Result:

- `xiuxian-qianji` test targets pass pedantic checks after cleanup.
- `xiuxian-qianji` all targets pass pedantic checks.
- `xiuxian-qianji` crate passes `too_many_lines` policy verification.

## Outcome

`xiuxian-qianji` advanced from failing `expect/unwrap`-denied test lanes to a
clean pedantic baseline for current touched paths, with no added broad lint
suppression.
