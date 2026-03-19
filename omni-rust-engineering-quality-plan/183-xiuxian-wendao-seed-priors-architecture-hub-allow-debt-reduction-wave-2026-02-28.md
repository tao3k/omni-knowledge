# 183. Xiuxian-Wendao Seed-Priors Architecture-Hub Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph_seed_and_priors/link_graph_structural_priors_promote_architecture_hub_top3.rs`

## Why This Wave

This seed-priors scenario leaf is isolated and suitable for continued
suppression removal with low change risk.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `link_graph_structural_priors_promote_architecture_hub_top3.rs`

2. Root-cause cleanup:
   - replaced `map_err(|e| e.to_string())` with `map_err(|e| e.clone())`
   - inlined assertion format args for score comparison message

No suppressions were reintroduced.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-wendao
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `14`
  - After this wave: `13`
  - Net reduction: `1` file

## Engineering Outcome

- Structural-priors architecture-hub scenario test is now suppression-free.
- Remaining debt is concentrated in a smaller set of seed-priors and larger
  CLI/agentic scenario files.
