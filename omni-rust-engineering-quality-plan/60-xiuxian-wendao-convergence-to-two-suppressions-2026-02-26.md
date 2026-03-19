# Xiuxian-Wendao Convergence To Two Suppressions (2026-02-26)

## Scope

This shard records the latest convergence pass that reduced
`xiuxian-wendao/src` clippy suppression debt to only two remaining entries.

Targets:

- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/types/commands/search.rs`
- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/execute/search.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph/relation_ops.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph/entity_ops.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph/errors.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/state.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/**/*.rs`
- Graph test updates for borrowed relation API.

## Changes Implemented

### 1) Removed `struct_excessive_bools` from CLI search args

Actions:

- Split `SearchArgs` bool flags into focused `clap(flatten)` sub-structs:
  - `SearchCaseOptions`
  - `SearchLinkToOptions`
  - `SearchLinkedByOptions`
  - `SearchFilterFlags`
  - `SearchVerbosityOptions`
- Updated search execution code to use nested flag paths.
- Preserved CLI option names and behavior.

### 2) Removed `struct_field_names` from parsed directive state

Actions:

- Renamed `ParsedDirectiveState` fields from `parsed_*` to concise names
  (for example `match_strategy`, `filters`, `limit_override`).
- Updated parse merge/scan/state call sites accordingly.
- Removed `#[allow(clippy::struct_field_names)]` from parse state module.

### 3) Propagated relation API signature refinement to tests

Actions:

- Updated graph tests to use borrowed relation calls after
  `add_relation(&Relation)` API alignment.

## Verification Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic
cargo test -p xiuxian-wendao --lib
```

Result:

- Library tests passed (`53/53`).

## Outcome

- `xiuxian-wendao/src` clippy suppression count reduced from `4` to `2`.
- Remaining suppressions:
  - `packages/rust/crates/xiuxian-wendao/src/entity/records.rs` (2x
    `struct_field_names`).
- Remaining items are domain-model naming decisions and should be handled with
  an explicit model/API migration plan if removal is required.
