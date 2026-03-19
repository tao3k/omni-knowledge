---
type: knowledge
metadata:
  title: "Trusted Source Repository URL Alignment"
  date: "2026-03-06"
  status: "completed"
---

# Trusted Source Repository URL Alignment (2026-03-06)

## Change

- Updated `packages/conf/settings.yaml` trusted source configuration from `github.com/omni-dev` to `github.com/tao3k/xiuxian-artisan-workshop`.

## Rationale

- The canonical repository URL is now `https://github.com/tao3k/xiuxian-artisan-workshop`.
- Security trusted-source defaults must match the real upstream identity to avoid stale allowlist behavior.

## Verification

Command:

```bash
rg --line-number --no-heading 'github\.com/omni-dev|omni-dev' . \
  --glob '!target/**' --glob '!.git/**' --glob '!.cache/**' \
  --glob '!.devenv/**' --glob '!.venv/**' --glob '!assets/knowledge/**'
```

Outcome:

- No active configuration references remain for `github.com/omni-dev`.
- Remaining `omni-*` hits are limited to preserved historical changelog entries.
