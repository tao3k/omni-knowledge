# LRU 0.12.5 Transitive Elimination Plan (2026-02-24)

## Objective

Eliminate `lru 0.12.5` from the `xiuxian-vector` dependency graph to remove
`RUSTSEC-2026-0002` while preserving search quality and performance baselines.

## Current Evidence

1. `xiuxian-vector` graph currently includes:
   - `tantivy 0.24.2` (via `lance 2.0.1` / `lance-index 2.0.1`)
   - `tantivy 0.25.0` (direct)
2. `lru 0.12.5` is pulled in through the mixed `tantivy` versions.
3. `cargo info` revalidation shows the workspace is already on current
   published `lance` and `tantivy` lines in this version set.

## Scope

- Primary crates:
  - `packages/rust/crates/xiuxian-vector`
  - `packages/rust/crates/omni-lance`
- Upstream chain:
  - `lance` / `lance-index`
  - `tantivy`

## Execution Strategy

1. Version-line convergence study:
   - Evaluate whether `xiuxian-vector` can align on a single `tantivy` line without
     breaking `lance` integration.
   - Document hard constraints where `lance` pins `tantivy 0.24.x`.

2. Optional backend boundary:
   - Split keyword backend adapter in `xiuxian-vector` so `tantivy` dependency
     surface is isolated behind a dedicated module boundary.
   - Prepare alternate backend path for controlled A/B validation.

3. Upstream pressure/fork fallback:
   - Track upstream `lance` migration path for non-affected `lru` chain.
   - If blocked, evaluate temporary internal patch branch for dependency
     override with full regression/perf validation.

4. Security exception cleanup:
   - After graph no longer resolves `lru 0.12.5`, remove
     `RUSTSEC-2026-0002` from:
     - `scripts/rust/cargo_audit_gate.sh`
     - `18-dependency-security-exception-register-2026-02-24.md`

## Validation Commands

```bash
# verify lru chain
cargo tree -p xiuxian-vector -e all | rg "lru v0\.12\.5|tantivy v0\.24\.2|tantivy v0\.25\.0"

# run targeted quality/perf gates
cargo test -p xiuxian-vector --test test_search_perf_guard
cargo test -p xiuxian-vector --test test_fusion_snapshots

# security gate
just rust-security-gate
# or
devenv tasks run ci:rust-security-gate
```

## Exit Criteria

1. `cargo tree -p xiuxian-vector -e all` no longer contains `lru v0.12.5`.
2. `RUSTSEC-2026-0002` removed from temporary exception list.
3. `xiuxian-vector` quality/perf guard tests remain green.
4. Rust security gate remains green.

## Owner

- Rust Vector Platform
