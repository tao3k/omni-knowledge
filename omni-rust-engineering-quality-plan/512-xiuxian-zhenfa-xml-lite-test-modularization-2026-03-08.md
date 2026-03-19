# 512. Xiuxian Zhenfa XML-Lite Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of `test_xml_lite.rs` in
`xiuxian-zhenfa`.

## Why This Change Was Needed

The XML-lite helpers are compact, but the workspace standard is that the top
integration-test file should be a launcher, not the implementation body. This
keeps the suite consistent with the rest of the Rust workspace.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-zhenfa/tests/test_xml_lite.rs` so it now
acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-zhenfa/tests/test_xml_lite/` with:

- `mod.rs` for the module graph only,
- `extract.rs` for tag extraction and numeric parsing assertions.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-zhenfa --tests
cargo nextest run -p xiuxian-zhenfa --no-fail-fast
cargo clippy -p xiuxian-zhenfa -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-zhenfa --tests` passed.
- `cargo nextest run -p xiuxian-zhenfa --no-fail-fast` passed (`32 passed, 0 skipped`).
- `cargo clippy -p xiuxian-zhenfa -- -W clippy::too_many_lines` passed.

Notes:

- `cargo check` emitted unrelated `missing-docs` warnings for
  `packages/rust/crates/xiuxian-zhenfa/tests/test_client.rs` and
  `packages/rust/crates/xiuxian-zhenfa/tests/test_gateway.rs`; both files were
  intentionally left untouched because they already contain in-flight user
  edits.

## Architectural Takeaways

- Even very small suites can use the same thin-launcher pattern without adding
  meaningful complexity.
- Consistent test entrypoints matter because they keep the workspace structure
  predictable across crates.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-zhenfa/tests/test_xml_lite.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_xml_lite/mod.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_xml_lite/extract.rs`
