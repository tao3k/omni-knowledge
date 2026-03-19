---
type: knowledge
metadata:
  title: "Pyproject Metadata Repository Alignment"
  date: "2026-03-06"
  status: "completed"
---

# Pyproject Metadata Repository Alignment (2026-03-06)

## Scope

This change aligned Python packaging metadata with the canonical repository identity `https://github.com/tao3k/xiuxian-artisan-workshop` and removed stale Omni-branded package descriptions.

## Changes Applied

- `pyproject.toml`
  - added `[project.urls]` with `Repository` and `Homepage`
- `packages/python/agent/pyproject.toml`
  - updated description to `Xiuxian Daochang - Thin MCP/Web Client Interface`
- `packages/python/core/pyproject.toml`
  - updated description to `Xiuxian Core - Kernel, Lifecycle, and Hot Reload`
- `packages/python/foundation/pyproject.toml`
  - updated description to `Xiuxian Foundation - Infrastructure Layer (AI, VCS, Text, Logging)`
- `packages/python/test-kit/pyproject.toml`
  - updated description to `Exclusive testing framework for Xiuxian Artisan Workshop`
- `packages/rust/bindings/python/pyproject.toml`
  - updated description to `Rust core bindings for Xiuxian Artisan Workshop`

## Verification

Command:

```bash
python - <<'PY'
import tomllib
paths = [
    'pyproject.toml',
    'packages/python/agent/pyproject.toml',
    'packages/python/core/pyproject.toml',
    'packages/python/foundation/pyproject.toml',
    'packages/python/test-kit/pyproject.toml',
    'packages/rust/bindings/python/pyproject.toml',
]
for path in paths:
    with open(path, 'rb') as handle:
        tomllib.load(handle)
print('ok')
PY
```

Outcome: all updated `pyproject.toml` files parse successfully.
