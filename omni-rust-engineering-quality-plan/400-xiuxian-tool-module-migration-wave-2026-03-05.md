# Xiuxian Tool Module Migration Wave (2026-03-05)

> **Category**: engineering-quality | **Date**: 2026-03-05

## Scope

This wave migrated the Python utility module naming from `omni_tool` to `xiuxian_tool`
while preserving the external tool command identity (`omni`).

## Changes

1. Renamed module file:
   - `packages/python/core/src/omni/core/omni_tool.py`
   - -> `packages/python/core/src/omni/core/xiuxian_tool.py`
2. Renamed API symbols:
   - `OMNI_TOOL_DESCRIPTION` -> `XIUXIAN_TOOL_DESCRIPTION`
   - `OMNI_INPUT_SCHEMA` -> `XIUXIAN_INPUT_SCHEMA`
   - `get_omni_tool_info()` -> `get_xiuxian_tool_info()`
   - `get_omni_tool_list_entry()` -> `get_xiuxian_tool_list_entry()`
3. Updated all in-repo imports and callsites:
   - `assets/skills/skill/scripts/list_tools.py`
   - `packages/python/agent/src/omni/agent/mcp_server/server.py`

## Compatibility

- The master gateway command name remains `omni`.
- This migration changes internal module/symbol naming only.

## Residual Audit Delta

- Before this wave: **91** matches
- After this wave: **86** matches
- Delta: **-5**

Audit command used:

```bash
rg --line-number --no-heading '\bomni-[a-z0-9-]+\b|\bomni_[a-z0-9_]+\b' . \
  --glob '!target/**' --glob '!.git/**' --glob '!.cache/**' \
  --glob '!.devenv/**' --glob '!.venv/**' --glob '!assets/knowledge/**'
```

## Verification Evidence

### Python syntax checks

```bash
uv run python -m py_compile \
  packages/python/core/src/omni/core/xiuxian_tool.py \
  assets/skills/skill/scripts/list_tools.py \
  packages/python/agent/src/omni/agent/mcp_server/server.py
```

Outcome: **PASS**.

### Runtime import smoke

```bash
uv run python - <<'PY'
from omni.core.xiuxian_tool import get_xiuxian_tool_info, get_xiuxian_tool_list_entry
from omni.agent.mcp_server.server import AgentMCPServer
print(get_xiuxian_tool_info().keys())
print(get_xiuxian_tool_list_entry()['command'])
server = AgentMCPServer(use_holographic=False)
print(type(server).__name__)
PY
```

Outcome: **PASS**.

### MCP handler regression (isolated pytest config)

```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest -c /dev/null -p pytest_asyncio.plugin \
  -o asyncio_mode=auto packages/python/agent/tests/unit/test_mcp_handler.py -q
```

Outcome: **PASS** (20 passed, 3 skipped).

## Rust Clippy Gate Note

No Rust crate source files were modified in this wave.
