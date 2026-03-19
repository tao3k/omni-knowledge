# 435. Xiuxian Wendao Markdown-Attachments Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the next Wendao LinkGraph test-architecture slice: migrating
`markdown_attachments` from inline corpus creation and piecemeal assertions to
fixture-backed `input/expected` contracts.

This slice keeps the same repository-wide direction as the previous Wendao test
migrations:

- corpus inputs are stored under `tests/fixtures/.../input/`,
- expected behavior is stored under `tests/fixtures/.../expected/`,
- test projection logic lives in a dedicated domain support module,
- no new top-level snapshot tree is introduced.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph/markdown_attachments.rs`
was still structured around repeated tempdir setup and direct field assertions.

That older shape had three costs:

- each scenario rebuilt its markdown corpus inline,
- link-extraction contracts were harder to inspect than the underlying fixture
  corpus,
- attachment-search behavior was validated through ad hoc assertions rather than
  a readable serialized contract.

## What Changed

### 1) Added a dedicated markdown-attachments fixture support module

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/markdown_attachments_fixture_support.rs`

This module owns:

- scenario materialization for markdown-attachment fixtures,
- stable JSON projection for stats plus neighbor rows,
- stable JSON projection for attachment-search hits,
- attachment-kind label normalization.

Why this matters:

- the suite now reads as pure behavior coverage,
- attachment output shaping is centralized,
- future attachment cases can reuse one explicit domain contract layer.

### 2) Migrated the full `markdown_attachments` suite to scenario fixtures

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/markdown_attachments.rs`

New scenario fixture trees:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/relative_and_anchor/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/reference_links/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/complex_markdown_links/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/ignore_attachments_and_inline_embeds/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/attachment_search_filters/...`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/relative_and_anchor/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/reference_links/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/complex_markdown_links/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/ignore_attachments_and_inline_embeds/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/attachment_search_filters/expected/result.json`

What the contracts now cover:

- relative markdown links plus anchor stripping,
- reference-style markdown links,
- complex markdown links resolved through Comrak parsing,
- attachment-link exclusion and inline-embed suppression,
- attachment search filtered by kind and extension.

Why this matters:

- attachment and markdown-link behavior now lives as readable scenario artifacts,
- corpus setup is no longer duplicated inline,
- future parser or attachment changes can be reviewed as JSON fixture diffs.

### 3) Extended the shared LinkGraph test root explicitly

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

What changed:

- imported the new markdown-attachments support module,
- registered it alongside the other domain fixture support surfaces.

Why this matters:

- the LinkGraph test root remains explicit,
- support continues to scale by domain instead of through generic helper sprawl.

## Architectural Takeaways

### Attachment extraction is a contract surface

The difference between navigable knowledge links and ignorable attachments is a
user-visible behavior boundary. It should be represented as fixture contracts,
not hidden in imperative test setup.

### Stable contracts should snapshot semantic fields, not ranking noise

This slice snapshots neighbor shape, link counts, attachment paths, kinds, and
extensions. It intentionally avoids attachment hit scores because the semantic
contract does not depend on exact scoring.

### A domain support module keeps test architecture clean

`markdown_attachments_fixture_support.rs` is a better long-term pattern than
expanding another shared helper file because ownership stays obvious from the
module name itself.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/markdown_attachments.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/markdown_attachments_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/relative_and_anchor/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/relative_and_anchor/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/relative_and_anchor/input/docs/sub/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/relative_and_anchor/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/reference_links/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/reference_links/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/reference_links/input/docs/sub/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/reference_links/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/complex_markdown_links/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/complex_markdown_links/input/docs/b(1).md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/complex_markdown_links/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/complex_markdown_links/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/ignore_attachments_and_inline_embeds/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/ignore_attachments_and_inline_embeds/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/ignore_attachments_and_inline_embeds/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/ignore_attachments_and_inline_embeds/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/attachment_search_filters/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/markdown_attachments/attachment_search_filters/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph markdown_attachments
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph markdown_attachments`
  passed (`5 passed, 79 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines`
  completed cleanly.

## Limits and Next Slice

The highest-value remaining assertion-heavy test module inside
`test_link_graph` is now:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`

That suite is the next natural migration candidate if we continue the same
fixture-first test-architecture program.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/434-xiuxian-wendao-graph-navigation-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/435-xiuxian-wendao-markdown-attachments-fixture-expected-contracts-2026-03-07.md`
