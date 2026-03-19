# 473. Xiuxian Wendao `test_enhancer` Modularization

Date: 2026-03-08

## Scope

This shard records the structural decomposition of
`packages/rust/crates/xiuxian-wendao/tests/test_enhancer.rs`.

## Why This Change Was Needed

The integration test entrypoint mixed four distinct concerns in one file:

- frontmatter parsing,
- relation inference,
- note enhancement,
- markdown config block and link extraction.

That layout obscured the real responsibility boundaries of the enhancer surface
and made future maintenance depend on a single mixed test namespace.

## What Changed

Reduced the root integration-test entrypoint to a thin launcher:

- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer.rs`

Created a dedicated directory module:

- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/frontmatter.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/relations.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/note_enhancement.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/markdown_config.rs`

Responsibility split:

- `frontmatter.rs`: frontmatter parsing coverage;
- `relations.rs`: inferred relation coverage;
- `note_enhancement.rs`: single-note and batch enhancement coverage;
- `markdown_config.rs`: config block extraction, memory index, and link-tracking coverage.

## Architectural Takeaways

- Parsing, enrichment, and config-extraction tests should not share a single
  mixed entrypoint just because they belong to one public module family.
- A directory module is the right shape when a test binary grows into several
  independent behavior slices with no shared fixture layer.
- Explicit per-slice test files make nextest output map directly to the domain
  under failure.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_enhancer --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_enhancer --no-fail-fast`
  passed (`15 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/frontmatter.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/relations.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/note_enhancement.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer/markdown_config.rs`
