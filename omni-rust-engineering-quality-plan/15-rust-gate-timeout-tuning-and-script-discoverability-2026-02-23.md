# Rust Gate Timeout Tuning And Script Discoverability (2026-02-23)

## Scope

Record the follow-up adjustments after Rust gate automation integration:

1. `rust-check` default timeout tuning.
2. discoverability updates for new Rust gate scripts.

## Changes

1. `justfile`
   - `rust-check` default timeout changed from `1800` to `3600`.
2. `scripts/README.md`
   - Added Rust gate scripts into the script catalog:
     - `scripts/rust/test_omni_core_rs.sh`
     - `scripts/rust/cargo_check_with_timeout.py`
   - Added quick command examples for both scripts.
3. Plan docs
   - Added the `just rust-check 3600` guidance in:
     - `omni-rust-engineering-quality-plan/README.md`
     - `12-rust-quality-gates-checklist-2026-02-23.md`

## Validation Evidence

1. `just --dry-run rust-check`
   - shows timeout default: `3600`.
2. `just rust-check 1`
   - exits with `124` (timeout guard works).
3. `python scripts/rust/cargo_check_with_timeout.py 2`
   - exits with `124` (direct timeout wrapper behavior verified).

## Operational Note

For busy local hosts, run:

```bash
just rust-check 3600
```

before full gate execution:

```bash
just rust-quality-gate
```
