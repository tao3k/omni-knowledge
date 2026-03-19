# Codex Rust Engineering Reference

## Purpose

This directory is a codex-only engineering reference based on
`.cache/researcher/openai/codex/`.

It captures reusable practices in Rust architecture, CI, security, and release
operations without project-specific implementation plans.

## Document Map

1. `00-research-method-and-scope.md`
   - Study method, source boundaries, and evidence rules.
2. `01-codex-engineering-patterns.md`
   - High-value engineering patterns extracted from codex sources.
3. `02-second-pass-codex-rust-engineering-deep-dive.md`
   - Second-pass deep dive on Codex Rust engineering management and runtime reliability.
4. `03-second-pass-codex-reliability-and-governance-expansion-2026-02-23.md`
   - Additional second-pass evidence on reliability contracts, CI governance, and release telemetry.
5. `04-third-pass-codex-rust-engineering-systems-patterns-2026-02-23.md`
   - Third-pass codex-only systems patterns across workspace policy, safety boundaries, and test reliability.
6. `05-fourth-pass-codex-rust-code-quality-patterns-2026-02-24.md`
   - Fourth-pass codex-only extraction focused on Rust code-quality engineering patterns
     (lint governance, error taxonomy, orchestration split, cancellation semantics, and hermetic tests).
7. `07-evidence-metrics-snapshot-2026-02-18.md`
   - Codex-only metrics and reproducible command fragments.

## How To Use

1. Read `00-research-method-and-scope.md` to understand what was measured.
2. Use `01-codex-engineering-patterns.md` as a pattern catalog.
3. Use `02-second-pass-codex-rust-engineering-deep-dive.md` for implementation-level evidence.
4. Use `03-second-pass-codex-reliability-and-governance-expansion-2026-02-23.md`
   for expanded reliability/governance evidence.
5. Use `04-third-pass-codex-rust-engineering-systems-patterns-2026-02-23.md`
   for the latest codex-only systems-level pattern extraction.
6. Use `05-fourth-pass-codex-rust-code-quality-patterns-2026-02-24.md`
   for code-quality-specific Rust engineering practices.
7. Validate assumptions against `07-evidence-metrics-snapshot-2026-02-18.md`.

## Maintenance Rules

- Keep this directory English-only.
- Keep this directory codex-only (no project-specific gap analysis or roadmap).
- Update metrics snapshot when codex source snapshots change.
