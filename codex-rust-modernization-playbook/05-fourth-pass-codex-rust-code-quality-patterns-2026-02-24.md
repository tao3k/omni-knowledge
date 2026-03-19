# Fourth-Pass Codex Rust Code-Quality Patterns (2026-02-24)

## Objective

Extract source-level Rust quality practices from Codex (`.cache/researcher/openai/codex/codex-rs`)
with a strict focus on code quality engineering, not project-specific migration work.

## Evidence Scope

Primary evidence files:

1. Workspace lint and profile policy:
   - `.cache/researcher/openai/codex/codex-rs/Cargo.toml:299`
   - `.cache/researcher/openai/codex/codex-rs/Cargo.toml:302`
2. Clippy policy tuning:
   - `.cache/researcher/openai/codex/codex-rs/clippy.toml:1`
3. Library output discipline:
   - `.cache/researcher/openai/codex/codex-rs/core/src/lib.rs:3`
4. Error model design:
   - `.cache/researcher/openai/codex/codex-rs/core/src/error.rs:63`
   - `.cache/researcher/openai/codex/codex-rs/core/src/unified_exec/errors.rs:4`
5. Runtime orchestration and cancellation:
   - `.cache/researcher/openai/codex/codex-rs/core/src/tools/orchestrator.rs:1`
   - `.cache/researcher/openai/codex/codex-rs/core/src/tools/parallel.rs:49`
6. Module ownership and decomposition:
   - `.cache/researcher/openai/codex/codex-rs/core/src/unified_exec/mod.rs:1`
7. Test determinism and hermeticity:
   - `.cache/researcher/openai/codex/codex-rs/core/tests/common/lib.rs:23`
   - `.cache/researcher/openai/codex/codex-rs/core/tests/suite/mod.rs:17`
   - `.cache/researcher/openai/codex/codex-rs/core/tests/all.rs:1`

## Codex Rust Quality Patterns

### 1) Strict Baseline, Narrow Exceptions

- Codex sets workspace-level deny rules for `unwrap_used` and `expect_used` and many
  manual-pattern lints at the workspace root.
- Exceptions are localized with `#[expect(...)]` / `#[allow(...)]` near the exact call site.
- This keeps the default quality bar high while allowing explicit, auditable escape hatches.

### 2) No Unstructured stdout/stderr in Library Crates

- Core crates deny direct `print_stdout` / `print_stderr`.
- This enforces output through explicit surfaces (TUI, protocol events, tracing).
- Result: fewer accidental side effects and cleaner testability.

### 3) Typed Error Taxonomy at Domain Boundaries

- Core defines domain error enums (`CodexErr`, `SandboxErr`, `UnifiedExecError`) with `thiserror`.
- Errors are categorized by actionability (retryability, rejection, invalid request, sandbox denial).
- Error types carry structured payloads when needed (for policy and UX handling).

### 4) Policy-Orchestration Isolated from Runtime Execution

- Tool approval/sandbox/retry flow is centralized in `tools/orchestrator.rs`.
- Runtime modules execute work under a passed-in attempt context, not global policy branching.
- This separation minimizes policy duplication and drift.

### 5) Concurrency and Cancellation as First-Class Design

- Tool execution paths model cancellation explicitly (`CancellationToken`, abort-on-drop handles).
- Parallelism policy is encoded with explicit locks and intent (`read` for parallel-capable, `write` for serialized tools).
- Span instrumentation is attached around async boundaries.

### 6) Module-Level Design Contracts

- Complex areas begin with module-level responsibility docs and flow summaries.
- `unified_exec/mod.rs` documents scope, flow, and internal split before implementation details.
- This improves maintainability under large-file realities.

### 7) Hermetic, Deterministic Test Harness

- Global test initialization (`#[ctor]`) sets deterministic process IDs and test-scoped environment.
- Tests avoid mutating developer real home/state by redirecting to temp directories.
- Integration tests are aggregated under a suite module with reusable helpers.

### 8) Quality Is Managed, Not Perfection Theater

- Codex still has large modules and selected lint suppressions in complex hotspots.
- Quality signal is not “zero exceptions”; it is “exceptions are explicit, localized, justified, and bounded.”

## Reusable Engineering Heuristics (Codex-Derived)

1. Keep lint policy centralized and strict; force exceptions to be local and reviewed.
2. Separate orchestration policy (approval/retry/sandbox) from tool runtime logic.
3. Prefer typed domain errors over stringly generic failures at subsystem boundaries.
4. Treat cancellation/abort semantics as required behavior, not optional polish.
5. Build deterministic test harness infrastructure early for long-term reliability.

## Notes

- This document is codex-only reference material.
- Project-specific gaps, rankings, and migration plans belong in
  `assets/knowledge/omni-rust-engineering-quality-plan/`.
