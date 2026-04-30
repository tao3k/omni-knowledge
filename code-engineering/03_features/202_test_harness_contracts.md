# Test Harness Contracts

:PROPERTIES:
:ID: code-engineering-test-harness-contracts
:PARENT: [[index]]
:TAGS: feature, tests, harness, determinism, isolation
:STATUS: ACTIVE
:END:

## Mission

A test harness is not just a wrapper around assertions. It is the execution
contract that makes validation deterministic, isolated, and reproducible.

## Mandatory Properties

- Deterministic identifiers and fixture inputs.
- Explicit environment setup.
- Minimal dependency surface.
- No hidden reliance on developer machine state.
- Stable failure messages that separate product regressions from environment
  blockers.

## Preferred Structure

1. Keep package-top or feature-top harness entrypoints explicit.
2. Mount only the fixtures and support helpers required by that lane.
3. Avoid cross-lane leakage through shared mutable global state.
4. Prefer real contract fixtures over synthetic placeholders when behavior is
   user-visible.

## Anti-Patterns

- Harnesses that depend on implicit path mounts.
- Snapshot lanes with unstable timestamps or random IDs.
- Mixed unit and integration semantics in the same opaque helper.
- Test-only compatibility shims that preserve bad structure indefinitely.

## Review Questions

- Can the lane run on a clean machine with only the declared environment?
- Does the failure tell us whether the problem is code, fixture drift, or
  missing prerequisites?
- Is the harness structure aligned with the package boundary it validates?

:RELATIONS:
:LINKS: [[01_core/101_engineering_harness_kernel]], [[03_features/201_tiered_verification_ladder]], [[03_features/204_environment_and_evidence_sync]]
:END:

---

:FOOTER:
:STANDARDS: v1.0
:END:
