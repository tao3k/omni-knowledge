# Zero-To-One Execution Plan (Feature-Based)

This roadmap is tracked by feature names.
Use week ranges only as planning hints.

## Feature 1: Rust Quality Gate Modernization

## Goal

Upgrade CI to enforce Rust quality, not just compilation.

## Scope

- Add dedicated clippy lane with `-D warnings`.
- Add `nextest` lane for faster and more reliable test execution.
- Keep existing targeted Rust tests for vector contracts and performance.

## Deliverables

1. Workflow updates in `.github/workflows/ci.yaml` and/or `checks.yaml`.
2. `just` tasks for local parity with CI.
3. CI summary output that clearly shows lint/test/security states.

## Acceptance

- PR with intentional clippy warning fails.
- Rust tests execute via `nextest` in CI.
- No regression to existing contract/perf gates.

## Target Window

Week 1 to Week 2.

## Feature 2: Dependency Security Guardrails

## Goal

Make supply-chain risk visible and enforceable.

## Scope

- Introduce `deny.toml`.
- Add `cargo-deny` and `cargo-audit` lanes.
- Define advisory exception policy.

## Deliverables

1. Security policy file at repo root.
2. CI workflow jobs for deny/audit.
3. Documentation for exception lifecycle and owners.

## Acceptance

- CI runs deny and audit on every PR.
- Every ignored advisory has reason and exit criteria.

## Target Window

Week 1 to Week 3.

## Feature 3: Workspace Lint Completion

## Goal

Ensure every Rust crate inherits workspace lint rules.

## Scope

- Add `[lints] workspace = true` to missing crates.
- Verify no crate bypasses workspace lint contract.

## Deliverables

1. Cargo manifest updates for missing crates.
2. Audit script/check integrated into CI.

## Acceptance

- 100% crates inherit workspace lints.
- CI check fails on new crates missing lint section.

## Target Window

Week 2.

## Feature 4: MCP Client Reliability Hardening

## Goal

Move `omni-mcp-client` from minimal viable client to robust production client.

## Scope

- Explicit lifecycle transitions and richer error taxonomy.
- Timeout and cleanup hardening.
- Integration tests for subprocess and network edge cases.

## Deliverables

1. Refined client state model in `packages/rust/crates/omni-mcp-client/src/client.rs`.
2. Test expansion under `packages/rust/crates/omni-mcp-client/tests`.
3. Runtime behavior notes in crate README.

## Acceptance

- Process/shutdown cleanup behavior is tested.
- Failure modes are deterministic and observable.

## Target Window

Week 2 to Week 4.

## Feature 5: Module Complexity Reduction

## Goal

Reduce hotspot maintenance risk in Rust and Python modules.

## Scope

- Split large files by domain concern.
- Remove `include!` composition in `omni-vector` runtime modules where practical.
- Break large Python runtime modules (especially MCP server and router/hybrid paths).

## Deliverables

1. Refactored module trees with stable public APIs.
2. Updated tests for moved code paths.
3. Lightweight architecture notes per subsystem.

## Acceptance

- Hotspot files reduced or clearly justified.
- No behavior regressions in existing test suites.

## Target Window

Week 3 to Week 7.

## Feature 6: Rust/Python Boundary Clarification

## Goal

Clarify ownership and contract boundaries across runtime layers.

## Scope

- Define single responsibility between:
  - `packages/python/mcp-server/src/omni/mcp/server.py`
  - `packages/python/agent/src/omni/agent/mcp_server/server.py`
- Document boundary contracts for payload schema and runtime responsibilities.

## Deliverables

1. Ownership map document per boundary.
2. Contract tests for critical cross-language payloads.
3. Cleanup/de-duplication plan for overlapping MCP server responsibilities.

## Acceptance

- No ambiguous ownership for MCP protocol handling and orchestration.
- Contract tests guard both sides of boundary.

## Target Window

Week 4 to Week 8.

## Feature 7: Release Engineering Modernization

## Goal

Increase release confidence for Rust artifacts and cross-platform behavior.

## Scope

- Add release preflight checks (version/tag coherence, core target builds).
- Add release workflow templates for Rust outputs.

## Deliverables

1. Release verification workflow(s).
2. Documented runbook for release failures and rollback.

## Acceptance

- Pre-release checks catch version and build issues before publish.

## Target Window

Week 7 to Week 10.

## Feature 8: Crate Documentation Completion

## Goal

Complete crate-level documentation baseline for maintainability.

## Scope

- Add README for crates currently missing one.
- Standardize README template:
  - purpose,
  - API surface,
  - test commands,
  - owner notes.

## Deliverables

1. README completion for all Rust crates.
2. CI check ensuring new crates include README.

## Acceptance

- 100% crate README coverage.

## Target Window

Week 6 to Week 10.

## Recommended Execution Order

1. `Rust Quality Gate Modernization`
2. `Dependency Security Guardrails`
3. `Workspace Lint Completion`
4. `MCP Client Reliability Hardening`
5. `Module Complexity Reduction`
6. `Rust/Python Boundary Clarification`
7. `Release Engineering Modernization`
8. `Crate Documentation Completion`

## Program-Level Exit Criteria

- Engineering quality gates are enforceable in CI.
- Dependency security checks are mandatory and maintained.
- Runtime hotspots are reduced or explicitly governed.
- Rust/Python boundary contracts are explicit and tested.
- Release process has reproducible verification.
