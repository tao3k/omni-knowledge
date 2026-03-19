# 393. Root renamer final convergence: `xiuxian-daochang` (2026-03-05)

## Goal

Close the root-level rename so the repository no longer contains legacy `omni-agent` naming in active source/config/tooling surfaces.

## Final Audit

- Residual grep (active repository surface):
  - command:
    - `rg --line-number --no-heading "\bomni-agent\b|\bomni_agent\b|packages/rust/crates/omni-agent" --glob '!assets/knowledge/**' --glob '!target/**' --glob '!.cache/**' --glob '!.venv/**' --glob '!.devenv/**' --glob '!.git/**'`
  - result:
    - no matches

- Positive grep for new namespace:
  - command:
    - `rg --line-number --no-heading "\bxiuxian-daochang\b|\bxiuxian_daochang\b|packages/rust/crates/xiuxian-daochang" --glob '!assets/knowledge/**' --glob '!target/**' --glob '!.cache/**' --glob '!.venv/**' --glob '!.devenv/**' --glob '!.git/**'`
  - result:
    - matches across Rust crate metadata, scripts, CI workflows, docs, and tests as expected

## Verification Evidence

- Rust package identity:
  - `cargo pkgid -p xiuxian-daochang` -> pass
  - `cargo pkgid -p omni-agent` -> expected failure (legacy package removed)

- Compile/lint/test gates:
  - `cargo check -p xiuxian-daochang -p xiuxian-wendao` -> pass
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` -> pass
  - `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` -> pass
  - `cargo nextest run -p xiuxian-daochang --test gateway_validation` -> `3 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-wendao --test test_wendao_cli` -> `34 passed`, `0 failed`

- Script/workflow surface sanity:
  - `bash scripts/ci_scripts_smoke.sh` -> pass (dry-run commands resolve to renamed script/task names)

## Outcome

- Root namespace is unified to `xiuxian-daochang` / `xiuxian_daochang` in active repository scope.
- Legacy `omni-agent` naming remains only in excluded generated or historical surfaces (`target`, virtualenv/cache, knowledge history), which is expected and non-runtime.
