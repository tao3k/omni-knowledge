---
type: knowledge
metadata:
  title: "Python Brand Residual Convergence Wave 2 (Low-Risk Internal Tokens)"
  date: "2026-03-05"
  status: "completed"
---

# Python Brand Residual Convergence Wave 2 (2026-03-05)

## Scope

This wave targeted low-risk Python-side and documentation brand residuals while preserving the live import namespace (`omni.agent`) to avoid runtime breakage.

## Changes Applied

1. Internal cache/schema identifiers were renamed to xiuxian-prefixed values.
   - `packages/python/core/src/omni/core/skills/tools_loader_index.py`
     - `_COMMAND_INDEX_CACHE_SCHEMA`: `omni.command-index.v1` -> `xiuxian.command-index.v1`
     - cache directory: `omni-tools-loader` -> `xiuxian-tools-loader`

2. Internal generator metadata was renamed.
   - `packages/python/agent/src/omni/agent/cli/commands/skill/generate.py`
     - `author`: `omni-rag-gen` -> `xiuxian-rag-gen`

3. Tracer checkpointer channel identifier was renamed.
   - `packages/python/foundation/src/omni/tracer/pipeline_checkpoint.py`
     - logger channel: `omni.tracer.pipeline` -> `xiuxian.tracer.pipeline`
     - checkpointer key: `omni_tracer_pipeline` -> `xiuxian_tracer_pipeline`

4. Bridge module inline contract text was aligned.
   - `packages/python/foundation/src/omni/foundation/bridge/__init__.py`
     - `import omni_rust_bindings` -> `import xiuxian_rust_bindings` (documentation text only)

5. Test/runtime temporary path labels were aligned.
   - `packages/python/foundation/tests/unit/services/test_vector_search_embedding_path.py`
     - `/tmp/omni-query-embed-test-*` -> `/tmp/xiuxian-query-embed-test-*`
   - `packages/python/agent/tests/unit/cli/test_app_runtime_paths.py`
     - `/tmp/omni-project` -> `/tmp/xiuxian-project`

6. Script user-agent token was aligned.
   - `scripts/fetch_previous_skills_benchmark_artifact.py`
     - `omni-skills-tools-baseline-fetcher` -> `xiuxian-skills-tools-baseline-fetcher`

7. Documentation owner/brand labels were aligned where low-risk.
   - `assets/specs/template.md`: `@omni-coder` -> `@xiuxian-coder`
   - `assets/specs/xml-qa-schema-augmentation.md`: `@omni-coder` -> `@xiuxian-coder`
   - `assets/specs/zettelkasten-knowledge-architecture.md`: `omni-rag` -> `xiuxian-rag`
   - `docs/01_core/omega/router.md`: `omni-router`/`xiuxian-vector`/`omni-embedding` textual references aligned to xiuxian variants
   - `docs/01_core/omega/trinity-control.md`: legacy removed module reference aligned (`omni.agent.cli.xiuxian_loop`)

## Verification Evidence

### Syntax validation

Command:

```bash
uv run python -m py_compile \
  packages/python/core/src/omni/core/skills/tools_loader_index.py \
  packages/python/agent/src/omni/agent/cli/commands/skill/generate.py \
  packages/python/foundation/src/omni/tracer/pipeline_checkpoint.py \
  packages/python/foundation/src/omni/foundation/bridge/__init__.py \
  packages/python/foundation/tests/unit/services/test_vector_search_embedding_path.py \
  packages/python/agent/tests/unit/cli/test_app_runtime_paths.py \
  scripts/fetch_previous_skills_benchmark_artifact.py
```

Outcome: pass (no syntax errors).

### Targeted tests

Commands and outcomes:

```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest -c /dev/null -p pytest_asyncio.plugin \
  packages/python/core/tests/units/test_tools_loader_index.py -q
# 2 passed

PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest -c /dev/null -p pytest_asyncio.plugin \
  packages/python/foundation/tests/unit/tracer/test_pipeline_checkpoint.py -q
# 3 passed

PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest -c /dev/null -p pytest_asyncio.plugin \
  packages/python/foundation/tests/unit/services/test_vector_search_embedding_path.py -q
# 19 passed

PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 uv run pytest -c /dev/null -p pytest_asyncio.plugin \
  packages/python/agent/tests/unit/cli/test_app_runtime_paths.py -q
# 1 passed
```

Notes:
- Warnings were limited to pytest marker registration and temporary cache path creation under `/dev`; no functional failures were observed.

## Residual Audit Snapshot

Residual scanner command:

```bash
rg --line-number --no-heading '\\bomni-[a-z0-9-]+\\b|\\bomni_[a-z0-9_]+' . \
  --glob '!target/**' --glob '!.git/**' --glob '!.cache/**' \
  --glob '!.devenv/**' --glob '!.venv/**' --glob '!assets/knowledge/**' | wc -l
```

Result: `55`

Primary remaining categories (intentionally deferred in this wave):
- External identity references (`github.com/omni-dev`, skill maintainer/author metadata).
- Historical changelog/research artifacts preserving original naming context.
- Project policy path naming (`assets/knowledge/omni-rust-engineering-quality-plan/`).
