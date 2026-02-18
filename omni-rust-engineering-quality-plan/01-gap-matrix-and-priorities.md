# Gap Matrix And Priorities

## Prioritization Rule

Prioritize by:

1. Risk reduction for production reliability.
2. Low-to-medium implementation cost.
3. High reuse across Rust and Python surfaces.

## Gap Matrix

| Area | Codex Pattern | Current Omni State | Impact | Priority |
| --- | --- | --- | --- | --- |
| Rust lint gate in CI | `clippy -D warnings` in CI matrix | no dedicated clippy gate | style drift, hidden warnings | P0 |
| Test runner scale | `cargo nextest` for speed and reliability | `cargo test` only | slower feedback, flaky handling weaker | P0 |
| Dependency security | `cargo-deny` + `cargo-audit` workflows | no equivalent lane | supply-chain blind spots | P0 |
| Crate lint inheritance | near-universal `[lints] workspace = true` | 5 crates missing | uneven rule enforcement | P0 |
| Output boundary safety | crate-level `deny(print_stdout/print_stderr)` | none in Rust crates | protocol/output regressions | P1 |
| MCP client hardening | explicit state machine + process cleanup tests | simplified client lifecycle | runtime edge-case risk | P1 |
| Module complexity governance | subsystem rules + strict practice | several >1k-line hotspots | maintainability and regression risk | P1 |
| Cross-language responsibility | explicit protocol boundary in Rust crates | dual MCP server stacks in Python | ownership ambiguity | P1 |
| Release discipline | tag/version checks + cross-target release workflows | limited release hardening | release fragility | P2 |
| Crate documentation hygiene | many module-level docs and guidance | several crates lack README | onboarding and review friction | P2 |

## Priority Buckets

## P0 Features (Start Immediately)

- Feature: `Rust Quality Gate Modernization`
- Feature: `Dependency Security Guardrails`
- Feature: `Workspace Lint Completion`

## P1 Features (Core Modernization)

- Feature: `MCP Client Reliability Hardening`
- Feature: `Module Complexity Reduction`
- Feature: `Rust/Python Boundary Clarification`

## P2 Features (Scale And Sustainability)

- Feature: `Release Engineering Modernization`
- Feature: `Crate Documentation Completion`

## Acceptance Signals Per Priority

## P0 Acceptance

- CI fails on Rust warnings.
- CI runs dependency security checks.
- All crates inherit workspace lints.

## P1 Acceptance

- MCP client handles lifecycle and cleanup failure cases with tests.
- Top hotspot files are split by domain concern.
- MCP server responsibilities are clearly partitioned and documented.

## P2 Acceptance

- Release lane validates version/tag consistency and major targets.
- Every crate has a minimal owner-facing README.
