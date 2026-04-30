# Code Engineering Knowledge Harness

## Purpose

This directory stores high-quality, modern code-engineering harness documents in
a Wendao-compatible knowledge layout.

It is intended for reusable engineering norms, verification rules, harness
contracts, and implementation governance that should remain queryable as a
knowledge graph rather than as a single flat note.

## Structure

- `index.md`
  - Map of content for this knowledge package.
- `02_languages/`
  - Language-specific constraints layered on top of the general harness.
- `01_core/`
  - Foundational engineering contracts and kernel rules.
- `03_features/`
  - Operational harness capabilities used during delivery.
- `05_research/`
  - Higher-level engineering principles and rationale.
- `06_roadmap/`
  - Planned expansion of the harness surface.

## Usage

1. Start from `index.md`.
2. Use `02_languages/` when a rule depends on a specific language.
3. Use `01_core/` for non-negotiable engineering invariants.
4. Use `03_features/` for implementation-time execution rules.
5. Use `05_research/` when a design decision needs deeper rationale.
6. Use `06_roadmap/` to track the next hardening slices.

## Maintenance Rules

- Keep all content English-only.
- Prefer stable identifiers and explicit cross-links.
- Keep documents small, queryable, and single-purpose.
- Add new norms as numbered documents under the correct tier instead of growing
  `index.md` into a monolith.
