# 392. Root Renamer: `omni-agent` -> `xiuxian-daochang` (audit + unification) (2026-03-05)

## Scope

- Objective: complete a root-level rename without compatibility leftovers.
- Source name family removed:
  - `omni-agent`
  - `omni_agent`
  - `packages/rust/crates/omni-agent`
- Target unified name family:
  - `xiuxian-daochang`
  - `xiuxian_daochang`
  - `packages/rust/crates/xiuxian-daochang`

## Audit Baseline

- Initial repository scan (excluding `assets/knowledge`):
  - `821` matches for `omni-agent|omni_agent` before this unification wave.
- Residual scan after migration (excluding build/cache/knowledge dirs):
  - `0` matches for `omni-agent|omni_agent|packages/rust/crates/omni-agent`.
  - scope exclusion rationale:
    - `assets/knowledge/**` is immutable historical evidence,
    - `target/**`, `.cache/**`, `.venv/**`, `.devenv/**` are generated/runtime artifacts.

## Implementation

1. Physical crate path rename:
   - moved `packages/rust/crates/omni-agent` -> `packages/rust/crates/xiuxian-daochang`.
   - updated workspace member path in root `Cargo.toml`.

2. Rust package + crate identity alignment:
   - package id remains `xiuxian-daochang`.
   - library crate id set to `xiuxian_daochang`.
   - removed all `use omni_agent::...`/`omni_agent::...` references across Rust crates/tests.

3. Command/runtime surface rename:
   - cargo package invocations changed from `-p omni-agent` to `-p xiuxian-daochang`.
   - binary invocations changed from `--bin omni-agent` to `--bin xiuxian-daochang`.
   - runtime log target defaults moved from `omni_agent` to `xiuxian_daochang`.

4. Tooling and CI rename:
   - renamed script file names:
     - `scripts/rust/omni_agent_*` -> `scripts/rust/xiuxian_daochang_*`
     - `scripts/channel/*omni-agent*` / `*omni_agent*` -> `*xiuxian-daochang*` / `*xiuxian_daochang*`
   - renamed workflow files:
     - `.github/workflows/omni-agent-*.yaml` -> `.github/workflows/xiuxian-daochang-*.yaml`
   - updated just/nix/workflow/script references to renamed files.

5. Python package metadata alignment:
   - updated distribution references from `omni-agent` to `xiuxian-daochang` in:
     - workspace `pyproject.toml`
     - `packages/python/agent/pyproject.toml`
     - `packages/python/test-kit/pyproject.toml`
     - `uv.lock`

## Verification

- Package identity:
  - `cargo pkgid -p xiuxian-daochang` -> pass
  - `cargo pkgid -p omni-agent` -> expected failure (package removed)

- Compile gates:
  - `cargo check -p xiuxian-daochang -p xiuxian-wendao` -> pass

- Mandatory clippy gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` -> pass

- Targeted runtime lane:
  - `cargo nextest run -p xiuxian-daochang --test gateway_validation` -> `3 passed`, `0 failed`

- Residual-name audit:
  - `rg "\bomni-agent\b|\bomni_agent\b|packages/rust/crates/omni-agent" ...` -> `0`

## Outcome

- The runtime name is now root-unified to `xiuxian-daochang` across code, crate metadata, scripts, CI, and docs/tooling references.
- No remaining old-name tokens in the audited repository scope.
- Build/test evidence confirms the renamed surface is executable.
