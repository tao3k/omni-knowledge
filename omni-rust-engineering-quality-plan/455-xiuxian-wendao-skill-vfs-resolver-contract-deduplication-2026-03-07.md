# 455. Xiuxian Wendao Skill VFS Resolver Contract Deduplication

Date: 2026-03-07

## Scope

This shard records the consolidation of resolver coverage for `xiuxian-wendao`
Skill VFS tests.

## Why This Change Was Needed

The repository carried two overlapping integration surfaces for resolver
behavior:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_resolver.rs`

The second file repeated behavior that was already better represented by the
fixture-backed `resolver_support_contract` in the first file:

- semantic root precedence,
- embedded mount gating,
- shared `Arc<str>` cache reuse,
- internal overlay precedence,
- runtime internal-root normalization.

That duplication fragmented the test story and left dead fixture directories in
place that existed only to support the redundant file.

## What Changed

### 1) Expanded the existing resolver support contract

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/expected/result.json`

The fixture-backed `resolver_support_contract` now also preserves the missing
resolver error semantics that were previously only asserted in the duplicate
resolver test file:

- missing semantic entity resolves to `ResourceNotFound`,
- missing internal namespace resolves to `UnknownInternalSkill`.

### 2) Removed the duplicate resolver integration test file

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_resolver.rs`

Resolver behavior now has a single contract-oriented home.

### 3) Removed dead fixture scenarios that only served the duplicate file

Removed fixture directories:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_semantic_uri/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_documents_and_manifests/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_overlay_prefers_first_root/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/overlay_precedence_by_root_order/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/shared_arc_internal/`

## Architectural Takeaways

- When fixture-backed contracts already express a behavior clearly, a second
  hand-authored integration file is usually technical debt, not extra safety.
- Test consolidation should include fixture consolidation. Keeping orphaned
  scenarios after deleting their only consumer is structure pollution.
- Error semantics belong in the same contract as successful resolution when
  they are part of the same public behavior surface.
- High-quality Rust testing is not just about more tests; it is about one clear
  authoritative test surface per behavior family.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_resolver.rs` (removed)
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_semantic_uri/` (removed)
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_documents_and_manifests/` (removed)
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_overlay_prefers_first_root/` (removed)
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/overlay_precedence_by_root_order/` (removed)
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/shared_arc_internal/` (removed)

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_skill_vfs_contracts --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ... --tests` completed cleanly.
- The full `test_skill_vfs_contracts` binary passed (`7 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Consolidated fixture root:
  - `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/455-xiuxian-wendao-skill-vfs-resolver-contract-deduplication-2026-03-07.md`
