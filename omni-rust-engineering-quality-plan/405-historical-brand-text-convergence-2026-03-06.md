---
type: knowledge
metadata:
  title: "Historical Brand Text Convergence"
  date: "2026-03-06"
  status: "completed"
---

# Historical Brand Text Convergence (2026-03-06)

## Scope

This wave cleaned the last editable historical `omni-*` brand text while intentionally leaving the knowledge-directory path unchanged.

## Changes Applied

- `CHANGELOG.md`
  - replaced historical repository-name text `omni-devenv-fusion` with `xiuxian-artisan-workshop`
- `docs/04_chronicles/research/2026-02-24-rust-embedding-stack-audit.md`
  - aligned temporary benchmark artifact labels from `omni_embed_bench` to `xiuxian_embed_bench`

## Verification

Command:

```bash
rg --line-number --no-heading 'omni-devenv-fusion|/tmp/omni_embed_bench|assets/knowledge/omni-rust-engineering-quality-plan' . \
  --glob '!target/**' --glob '!.git/**' --glob '!.cache/**' \
  --glob '!.devenv/**' --glob '!.venv/**' --glob '!assets/knowledge/**'
```

Outcome:

- `CHANGELOG.md` no longer contains `omni-devenv-fusion`
- the embedding audit no longer contains `omni_embed_bench`
- the only remaining matched path is `assets/knowledge/omni-rust-engineering-quality-plan/` in `AGENTS.md`, which is still the real on-disk directory path

## Follow-up Boundary

Renaming `assets/knowledge/omni-rust-engineering-quality-plan/` would require a physical directory move plus reference updates, so it was left for an explicit path-migration task.
