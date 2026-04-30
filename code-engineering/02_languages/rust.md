# Rust Constraints

:PROPERTIES:
:ID: code-engineering-rust-constraints
:PARENT: [[02_languages/index]]
:TAGS: language, rust, clippy, modularization, verification
:STATUS: ACTIVE
:END:

## Scope

These constraints apply when the touched surface is a Rust crate, Rust binding,
or Rust-owned verification lane.

## Structural Constraints

1. Prefer feature folders for medium and complex features.
2. Keep `mod.rs` interface-only.
3. Split by responsibility, not by file-size vanity alone.
4. Avoid hierarchical naming redundancy unless repetition resolves a real
   collision.
5. Default to `pub(crate)` for internal collaboration boundaries.

## Quality Constraints

1. Treat compiler and clippy warnings in the touched scope as work to close, not
   backlog.
2. Do not use global lint suppression as a substitute for design cleanup.
3. Add `# Errors` documentation for public `Result` APIs.
4. Prefer explicit types and error surfaces over convenience wrappers that hide
   failure semantics.

## Verification Constraints

1. Add or update tests for every behavior change.
2. Use narrow `cargo test` or equivalent pulse checks during iteration.
3. Run `cargo check` before widening the slice.
4. Run `cargo clippy --all-targets --all-features -- -D warnings` and
   `cargo nextest` before a feature is marked done.

## Environment Constraints

1. Prefer `direnv exec .` for project-scoped commands.
2. Treat missing binaries, lock contention, and storage exhaustion as
   environment blockers until proven otherwise.
3. Keep crate docs and execution records synchronized when the workflow requires
   both.

:RELATIONS:
:LINKS: [[02_languages/index]], [[01_core/101_engineering_harness_kernel]], [[03_features/201_tiered_verification_ladder]], [[03_features/203_module_boundary_and_naming]], [[03_features/204_environment_and_evidence_sync]]
:END:

---

:FOOTER:
:STANDARDS: v1.0
:END:
