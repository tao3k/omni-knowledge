---
type: knowledge
metadata:
  title: "Python Brand Residual Convergence Wave 3 (Safe Final Internal Sweep)"
  date: "2026-03-06"
  status: "completed"
---

# Python Brand Residual Convergence Wave 3 (2026-03-06)

## Scope

This wave finished the safe internal `omni-*` / `omni_*` residual cleanup without touching:

- external identity references,
- historical changelog/research artifacts,
- or the intentionally retained knowledge-path convention `assets/knowledge/omni-rust-engineering-quality-plan/`.

## Changes Applied

1. Python packaging guidance now matches the current MCP package identity.
   - `assets/instructions/project-conventions.md`
   - package name corrected from `omni_orchestrator` to `xiuxian-mcp`

2. Broken documentation references were replaced with live project documents.
   - `docs/testing/rust-agent-loop-memory-testing.md`
   - removed stale `../plans/omni-run-*` and `rust-agent-architecture-omni-vs-zeroclaw.md` links
   - replaced with current references:
     - `../04_chronicles/research/2026-02-25-rust-agent-hyperscale-runtime-plan.md`
     - `../03_features/session_governance.md`
     - `../01_core/agent/SPEC.md`

3. Internal Rust/Python fixture metadata was aligned with current project ownership.
   - `packages/rust/bindings/python/test_skill_index.json`
   - `authors: ["omni-dev"]` -> `authors: ["xiuxian-artisan-workshop"]`

## Validation Evidence

### Residual scanner

Command:

```bash
rg --line-number --no-heading '\\bomni-[a-z0-9-]+\\b|\\bomni_[a-z0-9_]+' . \
  --glob '!target/**' --glob '!.git/**' --glob '!.cache/**' \
  --glob '!.devenv/**' --glob '!.venv/**' --glob '!assets/knowledge/**'
```

Result: `9` residual matches remain.

Remaining matches are intentionally preserved and fall into three categories:

- external references: `github.com/omni-dev`
- historical records: `CHANGELOG.md`, benchmark artifact names under `docs/04_chronicles/research/2026-02-24-rust-embedding-stack-audit.md`
- policy path naming: `AGENTS.md` references to `assets/knowledge/omni-rust-engineering-quality-plan/`

### Targeted Rust verification

Command:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-brand-wave \
  cargo nextest run -p xiuxian-skills --test test_schema_validation test_skill_index_json_data_integrity
```

Outcome:

- `1 passed, 3 skipped`
- validated that the updated `packages/rust/bindings/python/test_skill_index.json` remains acceptable to the existing schema/data-integrity test lane

## Notes

- The default workspace `target/` was already locked by other cargo activity, so verification used an isolated temporary `CARGO_TARGET_DIR` to avoid cross-task interference.
- This wave intentionally avoided modifying external ecosystem identities that may still need to resolve to real upstream locations.
