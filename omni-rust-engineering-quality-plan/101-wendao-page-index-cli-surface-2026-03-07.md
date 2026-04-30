# Wendao PageIndex CLI Surface

## Summary

Extended the `wendao` CLI with a dedicated `page-index` graph command so the new Rust-native `PageIndex` tree can be inspected without writing ad-hoc debug code.

This follow-up keeps the command aligned with the existing graph command surface:

- it participates in the normal `LinkGraphIndex` bootstrap path,
- it uses the same ambiguity handling pattern as `metadata` and `resolve`,
- it serializes through a CLI-local view model instead of leaking transport concerns into the core `PageIndexNode` type.

## Design Notes

### Command contract

New CLI form:

```text
wendao --root <DIR> page-index <stem-or-id-or-path>
```

Output shape:

- `query`: original alias input,
- `resolved`: canonical metadata row,
- `root_count`: number of root page nodes,
- `roots`: recursive page tree payload.

### Why a CLI-local view model

`PageIndexNode` stores `Arc<str>` and is intentionally an in-memory core type. The CLI now converts it into a serializable `PageIndexNodeView` plus `PageIndexMetaView` inside the binary layer.

This keeps core domain models free from output-format concerns and avoids enabling broader serde behavior just for transport.

### Ambiguity policy

The command intentionally does not trust the internal alias map when multiple notes share the same stem. It reuses `resolve_metadata_candidates` and emits the same `ambiguous_stem` payload shape already used by metadata-oriented commands.

## Files Updated

- `packages/rust/crates/xiuxian-wendao/src/bin/wendao.rs`
- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/execute.rs`
- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/execute/graph.rs`
- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/execute/graph/page_index.rs`
- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/types.rs`
- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/types/commands/graph.rs`
- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/types/commands/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/cli_commands/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/cli_commands/page_index_command.rs`

## Validation Evidence

1. `cargo check -p xiuxian-wendao`
   - Result: passed.

2. `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
   - Result: passed.

3. `cargo nextest run -p xiuxian-wendao --test test_wendao_cli page_index`
   - Result: passed.
   - Covered cases:
     - hierarchical output for a normal note,
     - ambiguity reporting when two documents share the same stem.

## Follow-Up Opportunities

1. Add a matching Python binding surface so the same `PageIndex` tree is available to higher-level runtime code.
2. Add a `--pretty` example to user-facing Wendao docs once the command surface stabilizes.
3. Consider adding optional depth limiting or subtree extraction if very large documents become common.
