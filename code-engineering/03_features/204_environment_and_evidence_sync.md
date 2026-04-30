# Environment And Evidence Sync

:PROPERTIES:
:ID: code-engineering-environment-evidence-sync
:PARENT: [[index]]
:TAGS: feature, environment, docs, evidence, reproducibility
:STATUS: ACTIVE
:END:

## Purpose

High-quality engineering requires both execution parity and documentation
parity. A change is only trustworthy when the environment and the written record
match the real implementation state.

## Environment Rules

1. Prefer project-scoped commands through the exported project environment.
2. Use project path variables instead of hardcoded local paths.
3. Treat missing tools, lock contention, and storage exhaustion as environment
   blockers until proven otherwise.
4. Record environment prerequisites before escalating product-level concerns.

## Evidence Sync Rules

1. Update package-level documentation when structural behavior changes.
2. Keep daily execution tracking in sync with package docs when the workflow
   requires both.
3. Use ExecPlans for complex features or significant refactors.
4. Keep knowledge documents small, linked, and queryable.

## Completion Test

A delivery slice is synchronized only when:

- the repository contains the code change,
- the relevant verification lane has been executed or the blocker is recorded,
- the package-facing documentation reflects the new state, and
- the knowledge surface exposes stable retrieval anchors.

:RELATIONS:
:LINKS: [[01_core/101_engineering_harness_kernel]], [[03_features/201_tiered_verification_ladder]], [[05_research/301_modern_engineering_harness_principles]]
:END:

---

:FOOTER:
:STANDARDS: v1.0
:END:
