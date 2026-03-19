# 243 - `omni-tui` and `xiuxian-vector` `src` Path-Mount Elimination Wave - 2026-03-02

## Scope

This wave removes the last `src`-side test path mounts in `omni-tui` and
`xiuxian-vector`, then stabilizes package-top harnesses without adding suppression
attributes.

## Structural Changes

### Removed `src` test path-mounts

- `packages/rust/crates/omni-tui/src/main.rs`
- `packages/rust/crates/xiuxian-vector/src/keyword/fusion/match_util.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/mod.rs`

### Added package-top harnesses

- `packages/rust/crates/omni-tui/tests/main_demo_unit.rs`
- `packages/rust/crates/xiuxian-vector/tests/keyword_fusion_match_util_unit.rs`
- `packages/rust/crates/xiuxian-vector/tests/search_impl_unit.rs`

### Supporting refactor in `omni-tui`

- Extracted CLI argument model into:
  `packages/rust/crates/omni-tui/src/cli_args.rs`
- Updated `main.rs` to import `Args` from `cli_args` module.
- This avoids dead-code warnings from including full `main.rs` in test harnesses
  and keeps argument parsing tests tied to production definitions.

## Validation Evidence

### Mandatory strict clippy for touched crates

```bash
cargo clippy -p omni-tui -p xiuxian-vector -- -W clippy::too_many_lines
```

Result: success.

### Targeted `nextest` proofs

```bash
cargo nextest run -p omni-tui --test main_demo_unit
cargo nextest run -p xiuxian-vector --test keyword_fusion_match_util_unit --test search_impl_unit
```

Results:

- `omni-tui/main_demo_unit`: `3 passed`, `0 failed`
- `xiuxian-vector` two harnesses: `10 passed`, `0 failed`

## Burndown Status

Global `src` path-mount scan:

```bash
rg --line-number --glob 'packages/rust/crates/*/src/**/*.rs' '#\[path\s*=\s*"[^"]*tests/[^"]*"\]' | wc -l
```

Current remaining count: `15`.

Remaining mounts are now concentrated in `xiuxian-daochang` only.

## Outcome

`omni-tui/src` and `xiuxian-vector/src` are now path-mount zero and aligned with
the package-top test-entry standard, with strict clippy and targeted nextest
evidence preserved.
