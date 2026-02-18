# Omni Rust Engineering Quality Plan

## Purpose

This directory contains the project-specific modernization and execution plan
for `omni-dev-fusion`.

Unlike the codex reference directory, this folder is dedicated to:

- current-state diagnosis of this repository,
- prioritized implementation roadmap,
- execution checklists and tracking,
- objective verification of high-quality Rust engineering outcomes.

## Document Map

1. `00-omni-current-state-baseline.md`
   - Evidence-based baseline for current Rust/Python engineering state.
2. `01-gap-matrix-and-priorities.md`
   - Gap matrix and priority ordering.
3. `02-modern-rust-and-software-standards.md`
   - Target standards to enforce.
4. `03-zero-to-one-execution-plan.md`
   - Feature-based modernization roadmap.
5. `04-operating-checklists.md`
   - Daily/PR/CI/release execution checklists.
6. `05-evidence-metrics-snapshot-2026-02-18.md`
   - Reproducible metrics snapshot for this repository.
7. `06-high-quality-rust-engineering-scorecard.md`
   - Tracking framework to verify progress toward high-quality outcomes.

## How To Use

1. Start from `06-high-quality-rust-engineering-scorecard.md` and
   `00-omni-current-state-baseline.md`.
2. Execute features in `03-zero-to-one-execution-plan.md` using canonical
   feature names.
3. For each feature update, attach evidence (PR links, commands, test results)
   to the scorecard and snapshot files.

## Maintenance Rules

- Keep this directory English-only.
- Keep tracking feature-based and evidence-driven.
- Update snapshot and scorecard whenever quality signals change.
