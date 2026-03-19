# Incremental Sync Policy API Alignment and Cross-Crate Validation (2026-02-26)

## Scope

Finalize the second-wave validation after changing
`IncrementalSyncPolicy::new` to a borrowed slice API and remove downstream
mismatches without introducing suppression-based fixes.

## Implemented Changes

1. Cross-crate API alignment:
   - `xiuxian-wendao/src/sync/incremental.rs`
     - kept constructor as `pub fn new(extensions: &[String]) -> Self`
     - kept `from_glob_patterns` aligned with borrowed extension flow
     - kept nested branch collapse for glob brace parsing
2. Downstream compile fix:
   - `xiuxian-zhixing/tests/test_wendao_indexer.rs`
     - replaced direct `Vec<String>` call sites with borrowed slice usage:
       `let configured_extensions = vec![...];`
       `IncrementalSyncPolicy::new(&configured_extensions);`
3. Prior warning cleanup from this wave remains in place:
   - `xiuxian-zhixing/src/wendao/indexer/documents.rs`
     - retained `needless_continue` cleanup by directly propagating
       `self.sync_document_path(...)?;`

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-wendao -p xiuxian-zhixing -p xiuxian-daochang
cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic
cargo clippy -p xiuxian-zhixing --all-targets -- -W clippy::pedantic
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
cargo clippy -p xiuxian-zhixing -- -W clippy::too_many_lines
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -l "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests
```

Result:

- `xiuxian-wendao` pedantic check: pass.
- `xiuxian-zhixing` pedantic check: initial failure exposed two constructor
  mismatches in `test_wendao_indexer`; after call-site alignment, re-run pass.
- `xiuxian-wendao` and `xiuxian-zhixing` `too_many_lines` checks: pass.
- `xiuxian-daochang` test-target compilation and pedantic check: pass.
- `xiuxian-daochang/tests` marker scan remains empty (zero matched files).

## Outcome

The incremental sync policy constructor migration is now consistent across
`xiuxian-wendao` and `xiuxian-zhixing`, with `xiuxian-daochang` quality baselines
remaining stable after cross-crate revalidation.
