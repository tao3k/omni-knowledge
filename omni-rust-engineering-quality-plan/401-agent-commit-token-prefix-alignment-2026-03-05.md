# Agent Commit Token Prefix Alignment (2026-03-05)

> **Category**: engineering-quality | **Date**: 2026-03-05

## Scope

Aligned the `agent-commit` temporary token file naming in `justfile` with Xiuxian branding.

## Change

- `justfile`:
  - `TOKEN_FILE="/tmp/.omni_commit_token"`
  - -> `TOKEN_FILE="/tmp/.xiuxian_commit_token"`

## Validation

```bash
rg --line-number --no-heading 'omni_commit_token|\.omni_commit_token|xiuxian_commit_token|\.xiuxian_commit_token' . \
  --glob '!target/**' --glob '!.git/**' --glob '!.cache/**' --glob '!.devenv/**' --glob '!.venv/**'
```

Outcome: only `.xiuxian_commit_token` remains in `justfile`.

## Residual Audit Delta

- Before this micro-wave: **86** matches
- After this micro-wave: **85** matches
- Delta: **-1**
