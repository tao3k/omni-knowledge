# Second-Pass Revalidation And Execution Update (2026-02-23)

## Objective

Revalidate the project-specific Rust modernization plan after:

1. second-pass Codex source reread, and
2. latest local cleanup progress in this repository.

This file is Omni-only.

## 1. Updated Baseline (Current Repository)

## Workspace governance

- Workspace members: `23`
- Members with `[lints] workspace = true`: `23/23`
- Lint inheritance policy is complete at manifest level.

## Boundary safety policy

- Library crates in workspace: `21`
- Library crates with root-level print boundary deny:
  `#![deny(clippy::print_stdout|print_stderr)]`: `0/21`
- This remains a high-priority gap versus transport-safe engineering practice.

## Structural complexity snapshot

- Rust source total: `159291` lines
- Rust files `>=1000` lines: `11`
- Top current hotspots:
  - `packages/rust/crates/xiuxian-skills/tests/tools_scanner.rs` (`1608`)
  - `packages/rust/crates/xiuxian-daochang/src/mcp_pool.rs` (`1536`)
  - `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs` (`1482`)
  - `packages/rust/crates/xiuxian-vector/src/skill/ops_impl.rs` (`1274`)
  - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram.rs` (`1259`)

## Strict clippy convergence progress

Verified on 2026-02-23:

- `cargo clippy -p omni-edit -- -D warnings` -> `EXIT:0`
- `cargo clippy -p omni-tags -- -D warnings` -> `EXIT:0`
- `cargo clippy -p xiuxian-qianhuan -- -D warnings` -> `EXIT:0`
- `cargo clippy -p omni-memory -- -D warnings` -> `EXIT:0`
- `cargo clippy -p omni-executor -- -D warnings` -> `EXIT:0`

Current primary blocker after re-probe:

- `cargo clippy -p xiuxian-wendao -- -D warnings` -> `192` errors
  (after first quick-fix reduction from `202`).

Quality gate update already landed:

- `justfile` now runs workspace strict clippy in `rust-clippy`:
  `cargo clippy --workspace -- -D warnings`
- `rust-lint-inheritance-check` now validates
  `[lints] workspace = true` semantics.

## Current execution risk (environmental)

Some old local `rustc` processes entered uninterruptible `U` state during
earlier workspace-wide runs. This is an environment/runtime blocker, not a
design blocker, and can produce ambiguous workspace validation signals.

Action:

- Use isolated `CARGO_TARGET_DIR` and sequential crate checks until the host
  state is clean.

## 2. Refined Feature Execution (Next Iteration)

## Feature A: Full Workspace Strict Clippy (Completion)

Goal:

- Reach reliable `cargo clippy --workspace -- -D warnings` evidence.

Execution:

1. Run isolated-target strict checks sequentially by crate group.
2. Track first failing crate and fix before moving to next group.
3. Re-run full workspace strict command once host/process state is clean.

Exit:

- Full workspace strict command passes with reproducible logs.

## Feature C: Boundary Output Safety Rollout

Goal:

- Prevent accidental protocol/log contamination from library code paths.

Execution:

1. Pilot root-level print deny in transport/core libraries first.
2. Add explicit exceptions only at binary entrypoints.
3. Fix violations as code changes, not broad lint allows.

Exit:

- Boundary policy enabled in selected core libraries and enforced in CI.

## Feature D: Hotspot Modularization Wave 1

Goal:

- Reduce risk from multi-concern files in critical runtime paths.

Wave-1 targets:

1. `packages/rust/crates/xiuxian-daochang/src/mcp_pool.rs`
2. `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs`
3. `packages/rust/crates/xiuxian-vector/src/skill/ops_impl.rs`

Exit:

- Each target has concern-based submodules with behavior-preserving tests.

## Feature E/F: CI, Release, Security Hardening

Goal:

- Raise reliability and supply-chain confidence to modern baseline.

Execution:

1. Path-aware gating + required gatherer CI job.
2. Cargo timing and cache summary in CI outputs.
3. Mandatory `cargo-deny` and `cargo-audit` lanes with exception policy.

Exit:

- CI fails deterministically on lint/security regressions and provides
  actionable telemetry.

## 3. Seven-Day Execution Checklist

1. Reproduce full workspace strict clippy in clean host state.
2. Execute focused debt-reduction sprint for `xiuxian-wendao` (current primary blocker).
3. Land boundary-output deny pilot in one core library crate.
4. Submit first hotspot modularization PR (single target file).
5. Update scorecard and evidence snapshot with command outputs.

## 4. Canonical Verification Commands

```bash
cargo clippy --workspace -- -D warnings
cargo nextest run --workspace --no-fail-fast
just rust-lint-inheritance-check
just rust-clippy
```

Use command outputs from this set as required evidence anchors.
