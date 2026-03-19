# Brand Residual Convergence Wave (2026-03-05)

> **Category**: engineering-quality | **Date**: 2026-03-05

## Context

This wave continues the Omni -> Xiuxian convergence after crate/package-level renames.
The objective was to reduce low-risk naming residuals without changing public runtime contracts.

## What Was Updated

1. Documentation command examples switched from `@omni-orchestrator` to `@xiuxian-orchestrator`.
2. Internal variable/comment cleanup in Rust and Python where names still used `omni_*` even though behavior was Xiuxian-specific.
3. Temporary socket/path prefixes normalized from `omni-*` to `xiuxian-*` in benchmarks and tests.
4. Test fixture counter/temp naming cleanup (`omni_*` -> `xiuxian_*`) where behavior is unaffected.

## Residual Audit Delta

- Before this wave: **183** matches
- After this wave: **136** matches
- Delta: **-47**

Audit command used:

```bash
rg --line-number --no-heading '\bomni-[a-z0-9-]+\b|\bomni_[a-z0-9_]+\b' . \
  --glob '!target/**' --glob '!.git/**' --glob '!.cache/**' \
  --glob '!.devenv/**' --glob '!.venv/**' --glob '!assets/knowledge/**'
```

## Rust Quality Gate Evidence

### Clippy (required crates touched)

```bash
cargo clippy -p xiuxian-core-rs -p xiuxian-io -p xiuxian-macros -p xiuxian-qianhuan -p xiuxian-skills -p xiuxian-tui -- -W clippy::too_many_lines
```

Outcome: **PASS** (`Finished dev profile`, no blocking lint errors).

### Nextest

```bash
cargo nextest run -p xiuxian-macros
```

Outcome: **PASS** (16 passed, 0 failed).

```bash
cargo nextest run -p xiuxian-core-rs --test test_skill_index
```

Outcome: **PASS** (10 passed, 0 failed).

```bash
cargo nextest run -p xiuxian-io
```

Outcome: **PASS** (24 passed, 0 failed).

## Python Validation Notes

Targeted `pytest` runs are currently blocked by an environment/plugin registration conflict:

- `ValueError: Plugin already registered under a different name: omni_test_kit ...`
- Existing environment appears to load both `omni_test_kit` and `xiuxian_test_kit` entry points.

This issue is orthogonal to the naming changes in this wave and should be resolved in Python test environment/plugin config before using pytest as a gate.

## Next Recommended Convergence Targets

1. Decide whether `omni_cell` is a retained domain term or should migrate to `xiuxian_cell`.
2. Migrate remaining `omni_tool` naming only if a command-surface compatibility decision is made.
3. Keep external third-party references (`github.com/omni-dev/*`) unchanged unless upstream moves.
