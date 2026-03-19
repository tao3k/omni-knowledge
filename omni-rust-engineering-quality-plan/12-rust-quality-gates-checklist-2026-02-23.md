# Rust Quality Gates Checklist (2026-02-23)

## Purpose

Define the current high-quality Rust gate sequence for this repository with
reproducible commands and pass criteria.

## Gate Sequence (Local + CI)

## Gate 1: Formatting

Command:

```bash
cargo fmt --all --check
```

Pass criteria:

- Exit code is `0`.
- No formatting drift in Rust crates.

Recommended pre-step for heavy local environments:

```bash
just rust-check 3600
```

This uses the timeout-protected compile lane before running full quality gates.

## Gate 2: Strict Lint (Workspace)

Command:

```bash
cargo clippy --workspace -- -D warnings
```

Pass criteria:

- Exit code is `0`.
- No crate-level strict clippy regressions.

## Gate 3: Core Rust Test Baseline

Commands:

```bash
CARGO_TARGET_DIR=/tmp/workspace-strict-proof cargo test -p xiuxian-vector --no-fail-fast
scripts/rust/test_omni_core_rs.sh
scripts/rust/test_omni_core_rs.sh --lib --no-fail-fast
```

Pass criteria:

- Exit code is `0` for all commands.
- `xiuxian-vector` and `omni-core-rs` test contracts remain green.

## Gate 4: Rust-Python Bridge Contract Tests

Commands:

```bash
uv run pytest packages/python/foundation/tests/unit/services/test_vector_memory_guard.py -q
uv run pytest packages/python/foundation/tests/unit/services/test_rust_vector_bridge_schema.py -q
```

Pass criteria:

- Exit code is `0`.
- Python-side bridge contracts for Rust vector behavior remain stable.

## Gate 5: Security Lanes (Planned Mandatory)

Commands:

```bash
cargo audit
cargo deny check
```

Current status:

- Tracked in modernization Feature F.
- Keep command evidence in feature-progress snapshots until CI enforcement is
  fully wired.

## Gate 6: Full Workspace Regression Lane

Command:

```bash
cargo nextest run --workspace --no-fail-fast
```

Pass criteria:

- Exit code is `0`.
- Cross-crate regressions are blocked before release merge.

## Evidence Update Rule

After any gate changes or regressions:

1. Update score/trend details in
   `06-high-quality-rust-engineering-scorecard.md`.
2. Append command outcomes to the latest dated snapshot file in this directory.
