# Python Constraints

:PROPERTIES:
:ID: code-engineering-python-constraints
:PARENT: [[02_languages/index]]
:TAGS: language, python, typing, packaging, verification
:STATUS: ACTIVE
:END:

## Scope

These constraints apply when the touched surface is a Python package, adapter,
or Python-owned validation lane.

## Packaging Constraints

1. Treat `pyproject.toml` as the package contract.
2. Keep Python packages focused on adapter, connectivity, orchestration, or
   tooling boundaries rather than performance-critical kernel ownership.
3. Avoid hidden cross-package imports that bypass declared package boundaries.

## Quality Constraints

1. Prefer explicit types on public surfaces.
2. Keep data models and serialization contracts stable and reviewable.
3. Avoid dynamic behavior that obscures import-time side effects, environment
   dependencies, or runtime mutation of global state.
4. Keep scripts thin when the logic belongs in package modules.

## Verification Constraints

1. Use fast targeted tests during active iteration.
2. Run type checking before widening the change.
3. Keep formatting and linting clean in the touched scope.
4. Run the relevant `pytest` lane before a feature is marked done.

## Environment Constraints

1. Prefer the project environment through `direnv exec .`.
2. Use project-managed tooling such as `uv` where the workspace provides it.
3. Treat interpreter mismatch, dependency resolution failures, and missing
   optional services as environment blockers until proven otherwise.

:RELATIONS:
:LINKS: [[02_languages/index]], [[01_core/101_engineering_harness_kernel]], [[03_features/201_tiered_verification_ladder]], [[03_features/204_environment_and_evidence_sync]]
:END:

---

:FOOTER:
:STANDARDS: v1.0
:END:
