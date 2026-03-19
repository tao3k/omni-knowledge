# 275. xiuxian-llm Vision Scrub Pedantic Warning Cleanup (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-llm`
- Objective:
  - close newly surfaced pedantic warnings in the vision scrub policy lane,
  - preserve behavior with targeted regression proof,
  - keep zero-suppression remediation style.

## Changes

Updated:

- `packages/rust/crates/xiuxian-llm/src/llm/vision/scrub.rs`

Actions:

- replaced manual token membership check:
  - from `NOISE_TOKENS.iter().any(|token| normalized == *token)`
  - to `NOISE_TOKENS.contains(&normalized)`,
- replaced redundant closure:
  - from `text.chars().any(|ch| ch.is_alphanumeric())`
  - to `text.chars().any(char::is_alphanumeric)`.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-llm --test llm_vision
```

Result:

- `10 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0),
- no warning output for this crate in this slice.

## Outcome

- `vision/scrub.rs` no longer emits `manual_contains` or
  `redundant_closure_for_method_calls`,
- behavior remains stable under targeted tests,
- touched-crate pedantic/clippy gate is clean without lint suppression.
