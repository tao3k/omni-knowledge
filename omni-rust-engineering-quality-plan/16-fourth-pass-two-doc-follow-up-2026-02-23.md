# Fourth-Pass Two-Doc Follow-Up (2026-02-23)

## Objective

Continue execution using the two canonical inputs:

1. Codex reference:
   `assets/knowledge/codex-rust-modernization-playbook/04-third-pass-codex-rust-engineering-systems-patterns-2026-02-23.md`
2. Omni adoption plan:
   `assets/knowledge/omni-rust-engineering-quality-plan/13-third-pass-codex-to-omni-adoption-plan-2026-02-23.md`

This file is project-specific and tracks progress only for `omni-dev-fusion`.

## Slice Progress Snapshot

## Slice 1: Workspace Strict-Clippy Stability

Status: **Stable / Guarded**

Evidence:

- Workspace strict lint command exists and is mandatory in local gate entry:
  `justfile` (`cargo clippy --workspace -- -D warnings`).
- `rust-quality-gate` is wired in workflows:
  `.github/workflows/ci.yaml`, `.github/workflows/checks.yaml`.
- Lint inheritance scan result:
  22/22 crates include `[lints] workspace = true` (no missing crates).

Operational note:

- Keep this slice as no-regression policy, not a one-time cleanup task.

## Slice 2: Omni-Core-RS Runtime-Test Reliability

Status: **Integrated / Operational**

Evidence:

- Canonical runtime-safe launcher:
  `scripts/rust/test_omni_core_rs.sh`.
- Gate chain includes explicit library lane:
  `justfile` (`rust-test-omni-core-rs` target) and workflow jobs that call
  `ci:rust-omni-core-rs-lib` through devenv tasks.

Operational note:

- Keep script-based launch as the only supported entrypoint for this lane on
  macOS/Nix.

## Slice 3: Hotspot Modularization

Status: **In Progress / Still High Debt**

Current measurement:

- Rust source files > 1000 lines: **11**
- Rust source files > 500 lines: **46**

Top hotspots currently include:

- `packages/rust/crates/xiuxian-daochang/src/mcp_pool.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs`
- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs`

Operational note:

- This remains the main maintainability bottleneck despite lint convergence.

## Slice 4: Rust-Python Bridge Contract Tightening

Status: **In Progress / Good Test Surface**

Evidence:

- Bridge implementation:
  `packages/python/foundation/src/omni/foundation/bridge/rust_vector.py`
- Contract suites exist and are active:
  `packages/python/foundation/tests/unit/services/test_vector_memory_guard.py`
  and
  `packages/python/foundation/tests/unit/services/test_rust_vector_bridge_schema.py`
- Additional contract-oriented coverage exists:
  `packages/python/foundation/tests/unit/services/test_skills_rust_bridge_contract.py`

Operational note:

- This slice is no longer test-scarce; the next step is reducing duplicate
  contract logic and clarifying ownership boundaries.

## Slice 5: Dependency Security Lanes

Status: **Operational / Exception-Managed (Mandatory in CI)**

Current signal:

- Local security lane exists in `justfile`:
  `rust-security-audit`, `rust-security-deny`, `rust-security-gate`.
- Nix/CI task exists:
  `nix/modules/tasks.nix` -> `ci:rust-security-gate`.
- CI wiring exists in:
  `.github/workflows/ci.yaml` and `.github/workflows/checks.yaml`
  (rust security steps now mandatory).
- Baseline policy file added:
  `deny.toml`.
- P0 lockfile upgrades applied:
  `bytes`, `time`, `git2`, `oneshot`.
- Security gate execution now passes with explicit temporary exceptions:
  `devenv tasks run ci:rust-security-gate` -> pass.
- Current temporary exception set is tracked in:
  `17-dependency-security-lane-bootstrap-2026-02-24.md`.

Gap impact:

- Supply-chain risk now has active visibility and enforceable gate behavior.
  Remaining risk is explicitly bounded by temporary transitive exceptions.

## Slice 6: Release Engineering Hardening

Status: **Partial**

Current signal:

- Quality gates and test lanes exist.
- Formal release preflight controls (tag/version coherence, artifact validation
  policy) are not yet standardized as a first-class project lane.

## Execution Plan (Next Wave)

## Wave A (Immediate)

1. Security lane bootstrap (Slice 5).
   - Completed wiring in local gate + Nix + CI.
   - Completed first P0 advisory reduction wave.
   - Completed CI graduation from advisory to mandatory mode for rust security
     steps.
   - Next: reduce temporary exception set via:
     `19-reqwest011-transitive-decommission-plan-2026-02-24.md` and
     `20-lru0125-transitive-elimination-plan-2026-02-24.md`.
2. Hotspot decomposition kickoff (Slice 3).
   - Start with `xiuxian-daochang/src/mcp_pool.rs` and
     `xiuxian-vector/src/search/search_impl.rs`.
   - Enforce domain submodules + interface-only `mod.rs`.
3. Bridge contract ownership pass (Slice 4).
   - Group contract tests by boundary category (schema, fallback, cache, shape).
   - Remove duplicated assertions across files where possible.

## Wave B (After Wave A)

1. Release preflight lane (Slice 6).
   - Add version/tag/build coherence checks.
2. Scorecard update.
   - Recompute scores in
     `06-high-quality-rust-engineering-scorecard.md`
     after Wave A closes.

## Exit Criteria For This Follow-Up

This fourth-pass follow-up is considered complete when:

1. Slice 5 remains **Operational / Exception-Managed** and trends toward
   zero temporary exceptions.
2. Slice 3 line-count hotspot indicators trend downward in the next snapshot.
3. Slice 4 contract suites are grouped by ownership with reduced duplication.
4. Scorecard is updated with post-wave evidence links.
