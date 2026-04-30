# 397) Python Package Brand Unification (`omni-*` -> `xiuxian-*`)

Date: 2026-03-05
Scope: Python distribution package identity convergence across workspace metadata, dependency wiring, and lock files.

## Goal

Remove remaining `omni-*` distribution package names and unify on `xiuxian-*` while keeping runtime imports and tests functional.

## Renamed Python Distribution Packages

- `omni-core` -> `xiuxian-core`
- `omni-foundation` -> `xiuxian-foundation`
- `omni-test-kit` -> `xiuxian-test-kit`
- `omni-mcp` -> `xiuxian-mcp`

## Updated Surfaces

- Workspace root: `pyproject.toml`, `uv.lock`
- Package metadata:
  - `packages/python/core/pyproject.toml`
  - `packages/python/foundation/pyproject.toml`
  - `packages/python/test-kit/pyproject.toml`
  - `packages/python/mcp-server/pyproject.toml`
  - `packages/python/agent/pyproject.toml`
- Sources/dependency references in workspace configuration and docs/scripts were migrated from `omni-*` package names to `xiuxian-*` names.

Note:

- This wave renames Python **distribution names** only.
- Python module namespace (`omni.*`) was kept unchanged.

## Validation Evidence

### Dependency / lock coherence

- `uv lock --check`: PASS

### Runtime import sanity

- `uv run python -c "import xiuxian_core_rs; print('xiuxian_core_rs import ok')"`: PASS

### Targeted Python tests

- `uv run pytest packages/python/agent/tests/unit/session/test_session_window.py -q`: PASS (4/4)
- `uv run pytest packages/python/agent/tests/unit/cli/test_skill_sync.py -q`: PASS (13/13)

### Rust quality gate (touched bridge crate)

- `cargo clippy -p xiuxian-core-rs -- -W clippy::too_many_lines`: PASS

## Outcome

Python workspace package branding is now aligned with `xiuxian-*` for core/foundation/test-kit/mcp distributions, with lock file and targeted runtime/tests validated.
