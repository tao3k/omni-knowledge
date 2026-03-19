# 519. Xiuxian Daochang Hybrid Traceability Rendering

Date: 2026-03-08

## Scope

This shard records the downstream adoption of Wendao hybrid traceability fields
inside `xiuxian-daochang`.

## Why This Change Was Needed

`xiuxian-wendao` now exposes stable `doc_id` and `path` fields on the public
`QuantumContext` contract, but the Daochang-side hybrid renderer still emitted
semantic hits with only:

- `id`,
- `score`,
- a body containing the semantic path and anchor.

That meant the retrieval layer had already solved provenance, while the user-
facing XML-lite output still dropped the owning document and physical path.

During final validation, `clippy` also surfaced one concrete quality issue in
`xiuxian-daochang`'s CLI entrypoint: the gateway dispatch branch was still
creating an oversized future.

## What Changed

### Semantic Hybrid Rendering

Updated
`packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/wendao_search/render.rs`
so semantic hybrid hits now consume the richer `QuantumContext` contract.

The renderer now emits semantic `<hit>` records with stable provenance
attributes:

- `doc_id`
- `path`
- `type="semantic"`

The semantic body renderer was also extracted into one focused helper so the
formatted payload now includes:

- trace label,
- owning `doc_id`,
- physical `path`,
- outward compatibility `anchor_id`,
- related cluster labels when present.

### Contract Verification

Updated
`packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/wendao_search_tests.rs`
so the integration assertion now locks the downstream rendering contract for:

- semantic `type`,
- `doc_id="docs/alpha"`,
- `path="docs/alpha.md"`,
- body text containing the rendered provenance label.

### CLI Quality Cleanup

Updated `packages/rust/crates/xiuxian-daochang/src/main.rs` so the gateway
branch explicitly boxes `dispatch_gateway_command(...)` before awaiting it.

This preserves the existing behavior while satisfying `clippy`'s
`large_futures` signal during crate validation.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-daochang --tests
cargo nextest run -p xiuxian-daochang --test agent_zhenfa_unit --no-fail-fast
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-daochang --tests` passed.
- `cargo nextest run -p xiuxian-daochang --test agent_zhenfa_unit --no-fail-fast`
  passed (`20 passed, 1 skipped`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- A new public contract is not finished until downstream renderers and routers
  actually expose its value.
- Provenance should travel end-to-end through the retrieval stack instead of
  being reconstructed or silently dropped at the presentation boundary.
- Small async boxing at the command router boundary is preferable to carrying a
  known oversized future in the top-level CLI dispatch path.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/wendao_search/render.rs`
- `packages/rust/crates/xiuxian-daochang/src/main.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/wendao_search_tests.rs`
