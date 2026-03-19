# Rust Gate Automation Integration (2026-02-23)

## Scope

Integrate the `omni-core-rs` runtime-linking-safe test path into default Rust
quality gates for local and CI execution.

## Changes

1. `justfile`
   - Added `rust-test-omni-core-rs` target with pass-through arguments:
     - runs `scripts/rust/test_omni_core_rs.sh`
   - Added `rust-check` timeout control (`timeout_secs` argument, default
     `3600`) backed by:
     - `scripts/rust/cargo_check_with_timeout.py`
   - Standardized Rust quality-gate lanes to shared target dir:
     - `CARGO_TARGET_DIR=/tmp/workspace-strict-proof` default path in
       `rust-check`, `rust-clippy`, `rust-nextest`, `rust-test-omni-core-rs`
   - Updated `rust-quality-gate` dependencies to include:
     - `rust-test-omni-core-rs`
2. `.github/workflows/ci.yaml`
   - Updated Rust quality gate step label to reflect full gate scope
     (`strict clippy + nextest + omni-core-rs runtime lane`).
   - Added explicit `omni-core-rs --lib` runtime lane step.
   - Added explicit `CARGO_TARGET_DIR=/tmp/workspace-strict-proof` in test job
     environment for deterministic Rust gate pathing.
3. `.github/workflows/checks.yaml`
   - Updated Rust quality gate step label to reflect full gate scope
     (`strict clippy + nextest + omni-core-rs runtime lane`).
   - Added explicit `omni-core-rs --lib` runtime lane step.
   - Added explicit `CARGO_TARGET_DIR=/tmp/workspace-strict-proof` in
     rust-checks job environment.
4. `scripts/README.md`
   - Registered the new Rust gate scripts for team discoverability:
     - `scripts/rust/test_omni_core_rs.sh`
     - `scripts/rust/cargo_check_with_timeout.py`

## Operational Effect

- `just rust-quality-gate` now includes:
  1. lint inheritance check
  2. workspace check
  3. workspace strict clippy
  4. workspace nextest
  5. `omni-core-rs` runtime-safe test lane

This makes the `omni-core-rs` macOS/Nix runtime-linking guard part of the
default quality path instead of an optional manual step.
It also prevents unbounded local stalls in `rust-check` by enforcing a timeout
guard.

## Verification

Minimum verification commands:

```bash
just rust-test-omni-core-rs
just rust-test-omni-core-rs "--lib --no-fail-fast"
just rust-quality-gate
python scripts/rust/cargo_check_with_timeout.py 3600
```

Where full `rust-quality-gate` is too expensive during incremental work, run:

```bash
scripts/rust/test_omni_core_rs.sh
scripts/rust/test_omni_core_rs.sh --lib --no-fail-fast
```
