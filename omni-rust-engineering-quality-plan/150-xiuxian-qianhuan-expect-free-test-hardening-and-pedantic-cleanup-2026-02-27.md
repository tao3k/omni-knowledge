# Xiuxian-Qianhuan Expect-Free Test Hardening and Pedantic Cleanup (2026-02-27)

## Scope

Converge `xiuxian-qianhuan` test code with workspace lint policy:

1. Remove `expect()` usage in async integration/unit tests (workspace denies
   `clippy::expect_used`).
2. Resolve `clippy::manual_string_new` in hot-reload policy tests.
3. Revalidate with strict-clippy and targeted `cargo nextest`.

## Implemented Changes

1. Updated:
   - `packages/rust/crates/xiuxian-qianhuan/tests/test_xml_escape_hardening.rs`
   - Added `assemble_snapshot_or_panic` helper and replaced all
     `.expect(...)` usages with explicit `match`-based error handling.
2. Updated:
   - `packages/rust/crates/xiuxian-qianhuan/tests/unit_xml_validation.rs`
   - Added matching helper and replaced `.expect(...)` call sites.
3. Updated:
   - `packages/rust/crates/xiuxian-qianhuan/tests/test_hot_reload_policy.rs`
   - Replaced `"" .to_string()` pattern with `String::new()`.
4. Kept fixes suppression-free (no `#[allow(...)]` additions).

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-qianhuan
cargo clippy -p xiuxian-qianhuan --all-targets -- \
  -W clippy::pedantic -W clippy::too_many_lines
CARGO_TARGET_DIR=target/nextest-qianhuan cargo nextest run -p xiuxian-qianhuan \
  --test test_xml_escape_hardening \
  --test unit_xml_validation \
  --test test_hot_reload_policy
```

Results:

- Strict-clippy command completed successfully (exit `0`) after the fixes.
- Targeted `nextest` run completed successfully:
  - `12 tests run: 12 passed, 0 skipped`.

## Outcome

`xiuxian-qianhuan` test surfaces in this wave are now aligned with the
workspace no-`expect` standard and pedantic guidance, with executable evidence
for both lint and behavioral validation.
