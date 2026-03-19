# Xiuxian Cell Runtime Migration Wave (2026-03-05)

> **Category**: engineering-quality | **Date**: 2026-03-05

## Scope

This wave performed an atomic migration of the Python runtime module path and symbols from
`omni_cell` to `xiuxian_cell`.

### Core changes

1. Renamed module file:
   - `packages/python/core/src/omni/core/skills/runtime/omni_cell.py`
   - -> `packages/python/core/src/omni/core/skills/runtime/xiuxian_cell.py`
2. Renamed runtime class and related symbols:
   - `OmniCellRunner` -> `XiuxianCellRunner`
   - `_get_omni_cell` -> `_get_xiuxian_cell`
   - `_omni_cell` -> `_xiuxian_cell`
3. Updated all in-repo imports from:
   - `omni.core.skills.runtime.omni_cell`
   - -> `omni.core.skills.runtime.xiuxian_cell`
4. Renamed test files for consistency:
   - `packages/python/core/tests/units/runtime/test_omni_cell.py`
   - -> `packages/python/core/tests/units/runtime/test_xiuxian_cell.py`
   - `packages/python/core/tests/units/test_omni_cell_sys_query.py`
   - -> `packages/python/core/tests/units/test_xiuxian_cell_sys_query.py`
   - `assets/skills/omniCell/tests/test_omni_cell_modular.py`
   - -> `assets/skills/omniCell/tests/test_xiuxian_cell_modular.py`
5. Updated runtime metadata labels:
   - `omni-cell-rust` -> `xiuxian-cell-rust`
   - `omni-cell-fallback` -> `xiuxian-cell-fallback`

## Residual Audit Delta

- Before this wave: **136** matches
- After this wave: **91** matches
- Delta: **-45**

Audit command used:

```bash
rg --line-number --no-heading '\bomni-[a-z0-9-]+\b|\bomni_[a-z0-9_]+\b' . \
  --glob '!target/**' --glob '!.git/**' --glob '!.cache/**' \
  --glob '!.devenv/**' --glob '!.venv/**' --glob '!assets/knowledge/**'
```

## Verification Evidence

### Python syntax validation

```bash
uv run python -m py_compile \
  packages/python/core/src/omni/core/skills/runtime/xiuxian_cell.py \
  packages/python/agent/src/omni/agent/mcp_server/server.py \
  packages/python/agent/src/omni/agent/core/omni/react.py \
  packages/python/agent/src/omni/agent/core/evolution/factory.py \
  packages/python/agent/src/omni/agent/core/evolution/universal_solver.py \
  packages/python/agent/src/omni/agent/core/cortex/orchestrator.py \
  packages/python/agent/src/omni/agent/core/cortex/planner.py \
  packages/python/agent/src/omni/agent/workflows/robust_task/nodes.py \
  packages/python/agent/tests/units/test_evolution_solver.py \
  packages/python/core/tests/units/runtime/test_xiuxian_cell.py \
  packages/python/core/tests/units/test_xiuxian_cell_sys_query.py \
  packages/python/core/tests/units/test_async_loop_api.py \
  assets/skills/omniCell/scripts/nu_shell.py
```

Outcome: **PASS**.

### Targeted Python tests (isolated config)

To bypass the current repository-wide pytest plugin alias conflict during migration validation,
tests were executed with isolated pytest config (`-c /dev/null`) and explicit asyncio plugin loading.

```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest -c /dev/null -p pytest_asyncio.plugin \
  packages/python/core/tests/units/runtime/test_xiuxian_cell.py -q
```

Outcome: **PASS** (20 passed).

```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest -c /dev/null -p pytest_asyncio.plugin \
  packages/python/core/tests/units/test_xiuxian_cell_sys_query.py -q
```

Outcome: **PASS** (8 passed).

```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest -c /dev/null -p pytest_asyncio.plugin \
  packages/python/agent/tests/units/test_evolution_solver.py -q
```

Outcome: **PASS** (21 passed).

### Runtime smoke import

```bash
uv run python - <<'PY'
from omni.core.skills.runtime.xiuxian_cell import XiuxianCellRunner
from omni.agent.core.evolution.universal_solver import UniversalSolver
runner = XiuxianCellRunner()
print(runner.__class__.__name__)
print(UniversalSolver(trace_collector=None).__class__.__name__)
PY
```

Outcome: **PASS**.

## Rust Clippy Gate Note

No Rust crate source files were modified in this wave.
Therefore, Rust clippy gate was not applicable for this migration step.

## Remaining high-signal residuals

1. `omni-dev` (external upstream references)
2. `omni_tool` (Python tool gateway naming)
3. `omni_loop` (legacy decommission references and contract tests)

## Environment Note

Repository-default pytest invocation is still blocked by plugin alias duplication in the current
environment (`omni_test_kit` vs `xiuxian_test_kit`). This migration used isolated test execution
to validate code changes without mutating global pytest/plugin configuration.
