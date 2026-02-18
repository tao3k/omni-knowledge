# Research Method And Scope

## Objective

Build a codex-only engineering reference by studying
`.cache/researcher/openai/codex/` (especially `codex-rs`).

The output must be evidence-based, reusable, and maintainable across teams.

## Primary Source Scope

### Codex Sources

- Workspace and governance:
  - `.cache/researcher/openai/codex/codex-rs/Cargo.toml`
  - `.cache/researcher/openai/codex/codex-rs/clippy.toml`
  - `.cache/researcher/openai/codex/codex-rs/deny.toml`
  - `.cache/researcher/openai/codex/justfile`
  - `.cache/researcher/openai/codex/.github/workflows/rust-ci.yml`
  - `.cache/researcher/openai/codex/.github/workflows/cargo-deny.yml`
  - `.cache/researcher/openai/codex/codex-rs/.github/workflows/cargo-audit.yml`
- Core engineering and protocols:
  - `.cache/researcher/openai/codex/codex-rs/core/src/lib.rs`
  - `.cache/researcher/openai/codex/codex-rs/core/src/mcp_connection_manager.rs`
  - `.cache/researcher/openai/codex/codex-rs/rmcp-client/src/rmcp_client.rs`
  - `.cache/researcher/openai/codex/codex-rs/rmcp-client/tests/process_group_cleanup.rs`
  - `.cache/researcher/openai/codex/codex-rs/rmcp-client/tests/resources.rs`
- Process conventions:
  - `.cache/researcher/openai/codex/AGENTS.md`
  - `.cache/researcher/openai/codex/docs/contributing.md`

## Evaluation Dimensions

1. Workspace architecture and crate/module boundaries.
2. Lint and coding policy enforcement.
3. Test strategy and execution speed.
4. Dependency security and supply-chain governance.
5. Release engineering and platform coverage.
6. Runtime boundary safety and protocol reliability.
7. Operational discipline (docs, review rules, checklists).

## Method

1. Map codex workspace by directory and subsystem.
2. Compare governance artifacts (Cargo, CI, lint, release, docs).
3. Sample critical runtime paths (MCP client/server, config, protocol).
4. Capture quantitative metrics:
   - crate counts,
   - lint adoption rates,
   - large file distribution,
   - workflow coverage.
5. Build a reusable pattern set with concrete references.
6. Validate that each pattern is traceable to source evidence.

## Decision Rules Used In This Study

- Prefer evidence over intuition.
- Prioritize changes with high risk-reduction and low migration cost.
- Keep patterns concrete enough to be adopted and audited.
- Separate source-reference material from project-specific execution plans.

## Out Of Scope

- Rebuilding codex architecture 1:1.
- Project-specific gap analysis, roadmap, or backlog tracking.
- Recommendations without a concrete source reference.
