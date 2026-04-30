# Engineering Harness Kernel

:PROPERTIES:
:ID: code-engineering-harness-kernel
:PARENT: [[index]]
:TAGS: core, engineering, harness, governance
:STATUS: HARDENED
:END:

## Definition

The engineering harness is the deterministic control layer around day-to-day
software delivery. It defines the minimum rules that keep change velocity high
without sacrificing correctness, traceability, or maintainability.

## Kernel Invariants

1. Determinism before convenience.
2. Verification before completion claims.
3. Physical repository state before architectural theory.
4. Small focused modules before large mixed-responsibility files.
5. Evidence-backed documentation before folklore.

## Required Inputs

- The live repository state.
- The project execution environment.
- The target change boundary.
- The verification lane appropriate to the current stage.

## Expected Outputs

- A bounded change with explicit ownership.
- Updated verification artifacts or tests for touched behavior.
- A documented evidence trail for non-obvious decisions.
- Stable cross-links so knowledge retrieval does not depend on memory alone.

## Failure Modes

- Delivery proceeds from stale repository assumptions.
- Verification is skipped until the end of a large change wave.
- Naming and module trees encode implementation accidents instead of intent.
- Documentation drifts away from the actual package state.

:RELATIONS:
:LINKS: [[03_features/201_tiered_verification_ladder]], [[03_features/202_test_harness_contracts]], [[03_features/203_module_boundary_and_naming]], [[03_features/204_environment_and_evidence_sync]]
:END:

---

:FOOTER:
:STANDARDS: v1.0
:END:
