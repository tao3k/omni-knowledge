---
type: knowledge
metadata:
  title: "Pyproject Metadata Completion"
  date: "2026-03-06"
  status: "completed"
---

# Pyproject Metadata Completion (2026-03-06)

## Scope

This wave completed a modern baseline for package metadata across the workspace root, Python packages, Rust Python bindings, and skill-local Python packages.

## Metadata Standard Applied

Each relevant `pyproject.toml` now carries the following metadata where applicable:

- `readme`
- `requires-python`
- `authors`
- `license`
- `keywords`
- `classifiers`
- `project.urls.Repository`
- `project.urls.Homepage`
- `project.urls.Issues`

## Files Updated

- `pyproject.toml`
- `packages/python/agent/pyproject.toml`
- `packages/python/core/pyproject.toml`
- `packages/python/foundation/pyproject.toml`
- `packages/python/mcp-server/pyproject.toml`
- `packages/python/test-kit/pyproject.toml`
- `packages/python/xiuxian-wendao-py/pyproject.toml`
- `packages/rust/bindings/python/pyproject.toml`
- `skills/_template/pyproject.toml`
- `skills/crawl4ai/pyproject.toml`

## Additional Files Created

To support `readme = "README.md"` consistently, the following package-level README files were added:

- `packages/python/mcp-server/README.md`
- `packages/python/test-kit/README.md`
- `packages/rust/bindings/python/README.md`

## Verification

Command:

```bash
python - <<'PY'
import tomllib
from pathlib import Path
checks = {
    'pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers', 'dynamic'],
    'packages/python/agent/pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers', 'dependencies'],
    'packages/python/core/pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers', 'dependencies'],
    'packages/python/foundation/pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers', 'dependencies'],
    'packages/python/mcp-server/pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers', 'dependencies'],
    'packages/python/test-kit/pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers', 'dependencies'],
    'packages/python/xiuxian-wendao-py/pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers', 'dependencies'],
    'packages/rust/bindings/python/pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers'],
    'skills/_template/pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers', 'dependencies'],
    'skills/crawl4ai/pyproject.toml': ['readme', 'requires-python', 'authors', 'license', 'keywords', 'classifiers', 'dependencies'],
}
for path_str, required_keys in checks.items():
    path = Path(path_str)
    with path.open('rb') as handle:
        data = tomllib.load(handle)
    project = data['project']
    for key in required_keys:
        assert key in project, (path_str, key, sorted(project.keys()))
    urls = project['urls']
    assert urls['Repository'] == 'https://github.com/tao3k/xiuxian-artisan-workshop', path_str
    assert urls['Homepage'] == 'https://github.com/tao3k/xiuxian-artisan-workshop', path_str
    assert urls['Issues'] == 'https://github.com/tao3k/xiuxian-artisan-workshop/issues', path_str
    readme_path = path.parent / project['readme']
    assert readme_path.exists(), (path_str, readme_path)
print('ok')
PY
```

Outcome: all updated manifests parse successfully, expose the expected metadata fields, and point to an existing README file when `readme` is declared.

## Notes

- `src/omni` package paths and Python import namespaces were intentionally left unchanged. This wave only normalized packaging metadata.
- The workspace now exposes a consistent repository identity and packaging surface without changing runtime import compatibility.
