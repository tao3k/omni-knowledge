---
type: knowledge
metadata:
  title: "Subpackage Pyproject URL Alignment"
  date: "2026-03-06"
  status: "completed"
---

# Subpackage Pyproject URL Alignment (2026-03-06)

## Scope

This wave extended repository metadata alignment from the root `pyproject.toml` to subpackage and skill-local `pyproject.toml` files.

## Changes Applied

Added `[project.urls]` with the canonical repository URL to:

- `packages/python/agent/pyproject.toml`
- `packages/python/core/pyproject.toml`
- `packages/python/foundation/pyproject.toml`
- `packages/python/mcp-server/pyproject.toml`
- `packages/python/test-kit/pyproject.toml`
- `packages/python/xiuxian-wendao-py/pyproject.toml`
- `packages/rust/bindings/python/pyproject.toml`
- `skills/_template/pyproject.toml`
- `skills/crawl4ai/pyproject.toml`

Also updated:

- `skills/crawl4ai/pyproject.toml`
  - description text from `Omni` to `Xiuxian`

## Verification

Command:

```bash
python - <<'PY'
import tomllib
checks = {
    'packages/python/agent/pyproject.toml': ['dependencies', 'requires-python'],
    'packages/python/core/pyproject.toml': ['dependencies', 'requires-python'],
    'packages/python/foundation/pyproject.toml': ['dependencies', 'requires-python'],
    'packages/python/mcp-server/pyproject.toml': ['dependencies', 'requires-python'],
    'packages/python/test-kit/pyproject.toml': ['dependencies', 'requires-python'],
    'packages/python/xiuxian-wendao-py/pyproject.toml': ['dependencies', 'requires-python'],
    'packages/rust/bindings/python/pyproject.toml': ['classifiers', 'requires-python'],
    'skills/_template/pyproject.toml': ['dependencies', 'requires-python'],
    'skills/crawl4ai/pyproject.toml': ['dependencies', 'requires-python'],
}
for path, required_keys in checks.items():
    with open(path, 'rb') as handle:
        data = tomllib.load(handle)
    project = data['project']
    assert project['urls']['Repository'] == 'https://github.com/tao3k/xiuxian-artisan-workshop'
    assert project['urls']['Homepage'] == 'https://github.com/tao3k/xiuxian-artisan-workshop'
    for key in required_keys:
        assert key in project, (path, sorted(project.keys()), key)
print('ok')
PY
```

Outcome: all updated subpackage `pyproject.toml` files parse successfully, expose the canonical repository metadata, and keep required `project` keys attached to the main table rather than accidentally nesting them under `project.urls`.
