# Language-Specific Constraints

## Purpose

This directory stores language-scoped engineering constraints under the
`02_languages/` tier and refines the general code-engineering harness.

Use these documents when a rule depends on language semantics, toolchains, or
ecosystem-specific failure modes.

## Current Scope

- `index.md`
  - Language constraint map of content.
- `rust.md`
  - Rust-specific engineering constraints for this workspace.
- `python.md`
  - Python-specific engineering constraints for this workspace.

## Maintenance Rules

- Keep general rules in the package root tiers.
- Put only language-specific constraints here.
- Prefer one document per language instead of mixing multiple languages into one
  policy page.
