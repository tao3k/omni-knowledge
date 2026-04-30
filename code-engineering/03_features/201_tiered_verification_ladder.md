# Tiered Verification Ladder

:PROPERTIES:
:ID: code-engineering-tiered-verification
:PARENT: [[index]]
:TAGS: feature, verification, testing, delivery
:STATUS: ACTIVE
:END:

## Purpose

The verification ladder keeps iteration cheap during development while
preserving a strict completion gate for landed work.

## Tiers

### Tier 1: Pulse

Use the lightest consistency checks during active editing.

- Formatters.
- Fast narrow tests for the touched scope.
- Zero-warning checks in the touched lane when cheap to run.

### Tier 2: Heartbeat

Use broader compile-time and type-level validation before widening the change.

- `cargo check`
- type checking
- targeted integration or contract lanes

### Tier 3: Gate

Use the full industrial verification surface when the slice is done.

- `cargo clippy --all-targets --all-features -- -D warnings`
- `cargo nextest`
- any project-specific release or contract gates

## Operating Rules

1. Match verification cost to delivery stage.
2. Do not mark a feature complete before Gate passes.
3. Close warnings in the touched scope during the work, not as a deferred sweep.
4. When a verification lane is skipped for environmental reasons, record the
   blocker explicitly.

:RELATIONS:
:LINKS: [[01_core/101_engineering_harness_kernel]], [[03_features/202_test_harness_contracts]], [[03_features/204_environment_and_evidence_sync]]
:END:

---

:FOOTER:
:STANDARDS: v1.0
:END:
