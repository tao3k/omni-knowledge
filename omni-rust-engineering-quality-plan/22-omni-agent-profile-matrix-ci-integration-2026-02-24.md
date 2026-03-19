# 修仙道场 (Xiuxian Daochang) Profile Matrix CI Integration (2026-02-24)

## Objective

Promote `xiuxian-daochang` profile split validation from local-only evidence to
mandatory CI execution, covering both:

- default features,
- reduced profile (`--no-default-features`).

## Scope

- Task runner: `nix/modules/tasks.nix`
- Workflows:
  - `.github/workflows/ci.yaml`
  - `.github/workflows/checks.yaml`

## Implemented Changes

1. Added a dedicated Rust CI task:
   - Task id: `ci:rust-xiuxian-daochang-profiles`
   - File: `nix/modules/tasks.nix`
   - Command set:
     - `cargo check -p xiuxian-daochang`
     - `cargo check -p xiuxian-daochang --no-default-features`
     - `cargo test -p xiuxian-daochang --no-run`
     - `cargo test -p xiuxian-daochang --no-run --no-default-features`

2. Wired the task into mandatory workflow paths:
   - `.github/workflows/ci.yaml`
     - Added step:
       `Rust gate - xiuxian-daochang profile matrix (default + no-default-features)`.
   - `.github/workflows/checks.yaml`
     - Added step:
       `Rust gate - xiuxian-daochang profile matrix (default + no-default-features)`.

## Local Revalidation Evidence

Executed from repository root on 2026-02-24:

```bash
CARGO_TARGET_DIR=target/codex-agent-default-check cargo check -p xiuxian-daochang
CARGO_TARGET_DIR=target/codex-agent-nodflt-check cargo check -p xiuxian-daochang --no-default-features
CARGO_TARGET_DIR=target/codex-agent-default-check cargo test -p xiuxian-daochang --no-run
CARGO_TARGET_DIR=target/codex-agent-nodflt-check cargo test -p xiuxian-daochang --no-run --no-default-features
```

Result: all pass.

## Impact

1. Prevents silent regressions where reduced-profile compilation drifts behind
   default profile.
2. Converts feature-gate design from documentation-level intent to enforced
   CI policy.
3. Provides a stable precondition for the next phase in
   `19-reqwest011-transitive-decommission-plan-2026-02-24.md`.

## Next Slice

1. Dependency-graph assertion gate integration is completed in:
   - `23-xiuxian-daochang-dependency-graph-assertions-gate-2026-02-24.md`.
2. Continue LLM boundary decoupling from `litellm-rs` to native `reqwest 0.12`
   path while preserving behavior parity.
