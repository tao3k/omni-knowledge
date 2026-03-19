# Second-Pass Plan Refinement (2026-02-22)

## Objective

Refine the Rust engineering plan using a second-pass comparison against Codex
engineering patterns and the latest repository state.

This file is project-specific (Omni only).

For the latest revalidated baseline and execution updates, see
`08-second-pass-revalidation-and-execution-update-2026-02-23.md`.

## 1. Current Baseline (Second-Pass Snapshot)

## Workspace Policy Coverage

- Rust crates in workspace: `22`
- Crates with `[lints] workspace = true`: `22/22`
- Current lint inheritance policy is complete for crate manifests.

## Boundary Safety Signals

- Crates denying direct stdout/stderr prints at crate root: `0`
- This is a major gap relative to Codex boundary safety practice.

## Structural Complexity Snapshot

- Rust source total: `150402` lines
- Rust files with `>=1000` lines: `12`
- Representative hotspots:
  - `packages/rust/crates/xiuxian-skills/tests/tools_scanner.rs` (`1608`)
  - `packages/rust/crates/xiuxian-daochang/src/mcp_pool.rs` (`1536`)
  - `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs` (`1482`)
  - `packages/rust/crates/xiuxian-vector/src/skill/ops_impl.rs` (`1274`)
  - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram.rs` (`1259`)

## Strict Clippy Convergence (In-Progress)

- `omni-tui` is now clean under `cargo clippy -p omni-tui -- -D warnings`.
- Latest completed full-workspace strict run in this cleanup stream failed
  mainly in:
  - `omni-edit` (`28` previous errors)
  - `omni-tags` (`70` previous errors)
- Immediate modernization bottleneck is now concentrated and explicit.

## 2. V2 Feature Plan (Refined)

## Feature A: Strict Clippy Full Convergence

## Goal

Reach `cargo clippy --workspace -- -D warnings` with no crate exceptions.

## Scope

- Finish `omni-edit` lint backlog.
- Finish `omni-tags` lint backlog.
- Keep fixed crates stable via repeated workspace runs.

## Exit Criteria

- Full workspace strict clippy passes.
- No new crate-level broad lint allowances introduced.

## Feature B: Quality Gate Promotion (From Baseline Set To Full Workspace)

## Goal

Upgrade CI/local quality gate from selected crates to full workspace parity.

## Scope

- Update `just rust-clippy` to run workspace strict mode.
- Keep fast local lane separately if needed, but not as the only strict gate.
- Ensure CI gate command equals project standard.

## Exit Criteria

- `just rust-quality-gate` enforces workspace-wide strict clippy.
- CI "Rust quality gates" job fails on any crate regression.

## Feature C: Output Boundary Safety Contract

## Goal

Prevent accidental protocol/log corruption by separating core logic from direct
terminal output.

## Scope

- Add root-level `#![deny(clippy::print_stdout, clippy::print_stderr)]` to
  core library crates where appropriate.
- Document explicit exceptions for binary entrypoints and diagnostics.

## Exit Criteria

- Boundary lint policy exists in selected core crates.
- Violations are caught in CI before merge.

## Feature D: Hotspot Modularization Program

## Goal

Reduce maintenance risk in large modules by concern-based decomposition.

## Scope

- Prioritize top hotspots in `xiuxian-vector`, `xiuxian-daochang`, and `xiuxian-skills`.
- Extract domain-specific submodules; keep `mod.rs` as interface-only where
  modules are expanding.
- Move complex inline tests out of production modules as required by repo rules.

## Exit Criteria

- At least top five hotspots reduced or formally justified with follow-up issue.
- New module boundaries preserve public API and test behavior.

## Feature E: CI/Release Engineering Hardening

## Goal

Close the gap with Codex CI reliability and release confidence practices.

## Scope

- Introduce path-aware CI gating for expensive Rust jobs.
- Add Rust timing artifacts and cache observability where useful.
- Add release preflight checks (version/tag coherence + target build coverage).

## Exit Criteria

- CI includes deterministic required-result aggregation.
- Release preflight catches metadata/build mismatches before publish.

## Feature F: Dependency Security Lanes

## Goal

Make supply-chain checks mandatory and reviewable.

## Scope

- Add/complete `cargo-deny` and `cargo-audit` lanes.
- Define exception policy with owner and expiry.

## Exit Criteria

- Security checks run on each PR.
- Every exception has rationale and removal criteria.

## Feature G: Evidence-Driven Governance

## Goal

Track quality as measurable outcomes, not intent.

## Scope

- Update scorecard monthly.
- Keep snapshot files with reproducible commands.
- Attach evidence to each feature update.

## Exit Criteria

- Scorecard trend is visible and current.
- Feature status can be audited from evidence files.

## 3. Execution Sequence (Updated)

1. Feature A: Strict Clippy Full Convergence
2. Feature B: Quality Gate Promotion
3. Feature C: Output Boundary Safety Contract
4. Feature D: Hotspot Modularization Program
5. Feature F: Dependency Security Lanes
6. Feature E: CI/Release Engineering Hardening
7. Feature G: Evidence-Driven Governance

## 4. 30/60/90-Day Milestones

## Day 30

- `omni-edit` and `omni-tags` strict clippy backlog mostly cleared.
- Workspace strict clippy command wired into local quality gate draft.

## Day 60

- Workspace strict clippy is mandatory in CI.
- Boundary safety lint policy enabled for selected core libraries.
- First hotspot decomposition PR set merged.

## Day 90

- Security lanes and release preflight active.
- Scorecard reflects measurable movement toward high-quality gate definition.

## 5. Verification Commands

```bash
cargo clippy --workspace -- -D warnings
cargo nextest run --workspace --no-fail-fast
just rust-quality-gate
```

Use these commands as canonical evidence anchors in feature updates.
