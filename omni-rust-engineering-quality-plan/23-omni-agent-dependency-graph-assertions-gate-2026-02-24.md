# 修仙道场 (Xiuxian Daochang) Dependency Graph Assertions Gate (2026-02-24)

## Objective

Add a lightweight but enforceable dependency-graph gate for `xiuxian-daochang` to
ensure feature-gated profile isolation remains true in CI.

## Scope

- Task runner: `nix/modules/tasks.nix`
- Workflows:
  - `.github/workflows/ci.yaml`
  - `.github/workflows/checks.yaml`

## Gate Design

1. Hard assertions (fail CI on regression):
   - default profile must include `litellm-rs`,
   - `--no-default-features` profile must not include `litellm-rs`.

2. Advisory telemetry (printed for tracking, not failing yet):
   - count of `reqwest v0.11` matches in each profile graph,
   - count of `rustls-pemfile v1.x` matches in each profile graph.

This balances immediate regression protection with ongoing transitive cleanup
work from plan `19`.

## Implemented Changes

1. Added task:
   - Task id: `ci:rust-xiuxian-daochang-dependency-assertions`
   - File: `nix/modules/tasks.nix`
   - Uses `cargo tree -p xiuxian-daochang -e all` with and without
     `--no-default-features`.

2. Wired task into mandatory workflow paths:
   - `.github/workflows/ci.yaml`
     - Step:
       `Rust gate - xiuxian-daochang dependency graph assertions`.
   - `.github/workflows/checks.yaml`
     - Step:
       `Rust gate - xiuxian-daochang dependency graph assertions`.

## Local Revalidation Evidence

Equivalent assertions were executed locally on 2026-02-24.

Result:

```text
ASSERT_OK default:reqwest0.11=15 rustls1x=1 no-default:reqwest0.11=13 rustls1x=1
```

Interpretation:

1. Feature split is behaving as expected (`litellm-rs` excluded in reduced profile).
2. `reqwest 0.11`/`rustls-pemfile 1.x` still exist in both profiles, consistent
   with current `serenity` transitive path and plan `19` status.

## Impact

1. Turns profile-isolation assumptions into CI-enforced contracts.
2. Adds a reproducible dependency-signal baseline for transitive-removal work.
3. Reduces risk of silent feature wiring regressions while refactoring LLM and
   Discord integration boundaries.

## Next Slice

1. Start internal HTTP boundary extraction in `xiuxian-daochang` (plan `19`) so
   `litellm-rs` path is a compatibility adapter, not a core-path dependency.
2. Revisit `serenity` isolation strategy after boundary extraction reaches
   behavior-parity checkpoints.
