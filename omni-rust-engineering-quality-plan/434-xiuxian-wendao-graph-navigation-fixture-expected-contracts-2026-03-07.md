# 434. Xiuxian Wendao Graph-Navigation Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the next Wendao LinkGraph test-architecture slice: migrating
`graph_navigation` from inline corpus setup and direct assertions to
scenario-oriented `input/expected` fixtures.

This slice follows the same test-architecture contract used by the surrounding
Wendao migrations:

- corpus inputs live under `tests/fixtures/.../input/`,
- expected behavior lives under `tests/fixtures/.../expected/`,
- projection logic stays in a dedicated domain support module.

It does not introduce another snapshot root.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph/graph_navigation.rs`
was still a dense mixed-concern test file.

The previous shape bundled together:

- temporary fixture construction,
- traversal verification,
- metadata assertions,
- TOC assertions,
- PPR diagnostics checks.

That made the suite harder to scan and harder to evolve because the real
contract was spread across many small assertions instead of one readable
scenario artifact.

## What Changed

### 1) Added a dedicated graph-navigation fixture support module

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/graph_navigation_fixture_support.rs`

This module owns:

- scenario materialization for `graph_navigation` fixture trees,
- stable JSON projection for neighbor rows, related rows, metadata, and TOC
  documents,
- diagnostics projection for related-PPR metrics,
- a deliberate normalization step that keeps non-deterministic timing fields as
  semantic boolean invariants instead of fragile exact values.

Why this matters:

- traversal tests now read as behavior-only code,
- diagnostics contracts stay strict without binding to noisy runtime timing,
- support logic remains localized under a domain-specific module name.

### 2) Migrated the full `graph_navigation` suite to per-scenario fixtures

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/graph_navigation.rs`

New scenario fixture trees:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/neighbors_related_metadata_and_toc/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_with_diagnostics/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_from_seeds_with_diagnostics/...`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/neighbors_related_metadata_and_toc/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_with_diagnostics/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_from_seeds_with_diagnostics/expected/result.json`

What the contracts now cover:

- neighbor traversal surface,
- related-note surface,
- metadata extraction,
- TOC serialization,
- forced-subgraph related diagnostics,
- partitioned related-from-seeds diagnostics.

Why this matters:

- the full traversal contract now lives next to the corpus that drives it,
- diagnostics output is reviewable as structured JSON instead of hand-written
  assertion fragments,
- future topology changes can be audited with file diffs instead of re-reading
  test control flow.

### 3) Extended the shared LinkGraph test root explicitly

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

What changed:

- imported the new graph-navigation support surface,
- kept support registration explicit rather than relying on hidden module-local
  wiring.

Why this matters:

- the `test_link_graph` root stays predictable,
- fixture support continues to scale as domain-specific modules instead of a
  generic helper pile.

## Architectural Takeaways

### Traversal contracts are first-class behavior, not incidental internals

Neighbors, related rows, metadata, and TOC output all feed downstream retrieval
and debugging surfaces. They deserve explicit contract artifacts.

### Diagnostics should be normalized by semantic stability

For observability-style outputs, exact timing values are the wrong contract.
The better contract is whether metrics are present, bounded, positive, or
consistent. This slice locks those invariants while avoiding noisy flakes.

### The fixture tree is now the single source of truth for the scenario

Each graph-navigation scenario now carries both its corpus and expected output in
one place. That is the right long-term structure for a high-quality Rust test
surface.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/graph_navigation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/graph_navigation_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/neighbors_related_metadata_and_toc/input/root/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/neighbors_related_metadata_and_toc/input/root/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/neighbors_related_metadata_and_toc/input/root/sub/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/neighbors_related_metadata_and_toc/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_with_diagnostics/input/root/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_with_diagnostics/input/root/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_with_diagnostics/input/root/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_with_diagnostics/input/root/d.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_with_diagnostics/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_from_seeds_with_diagnostics/input/root/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_from_seeds_with_diagnostics/input/root/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_from_seeds_with_diagnostics/input/root/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_from_seeds_with_diagnostics/input/root/d.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_from_seeds_with_diagnostics/input/root/e.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_from_seeds_with_diagnostics/input/root/f.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/graph_navigation/related_from_seeds_with_diagnostics/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph graph_navigation
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph graph_navigation`
  passed (`3 passed, 81 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines`
  completed cleanly.

## Limits and Next Slice

The highest-value assertion-heavy suites still remaining in `test_link_graph`
are now:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/markdown_attachments.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`

`markdown_attachments.rs` is probably the better next step because it still
mixes inline file construction with contract-shape assertions around attachment
resolution.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/433-xiuxian-wendao-search-match-strategies-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/434-xiuxian-wendao-graph-navigation-fixture-expected-contracts-2026-03-07.md`
