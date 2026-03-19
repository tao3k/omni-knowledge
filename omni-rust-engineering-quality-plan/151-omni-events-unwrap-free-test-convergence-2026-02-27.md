# Omni-Events Unwrap-Free Test Convergence (2026-02-27)

## Scope

Enforce workspace unwrap policy in `omni-events` while keeping behavior
unchanged:

1. Remove `unwrap()` usage from async broadcast tests in `omni-events`.
2. Revalidate with strict-clippy (`pedantic` + `too_many_lines`).
3. Execute crate tests with `cargo nextest`.

## Implemented Changes

1. Updated:
   - `packages/rust/crates/omni-events/src/lib.rs`
2. Added test-local helper:
   - `recv_or_panic(&mut broadcast::Receiver<OmniEvent>) -> OmniEvent`
3. Replaced three `rx.recv().await.unwrap()` call sites with helper calls in:
   - `test_event_bus_publish`
   - `test_multiple_subscribers`
4. No suppression attributes were introduced.

## Verification Evidence

Executed:

```bash
cargo fmt -p omni-events
cargo clippy -p omni-events --all-targets -- \
  -W clippy::pedantic -W clippy::too_many_lines
CARGO_TARGET_DIR=target/nextest-omni-events cargo nextest run -p omni-events
```

Results:

- Strict-clippy command completed successfully (exit `0`).
- `cargo nextest` completed successfully:
  - `5 tests run: 5 passed, 0 skipped`.

## Outcome

`omni-events` test code in this wave now conforms to the workspace no-`unwrap`
rule via explicit error handling, with lint and runtime test evidence.
