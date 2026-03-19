# 213. Zhixing Resource Image Export and Indexer Test Modularization Wave (2026-03-01)

## Scope

- Export a canonical embedded resource image from `xiuxian-zhixing`.
- Remove legacy `embed_utf8_dir!` usage from `xiuxian-daochang` Zhixing bootstrap path.
- Add integration proof that Wendao can resolve `SKILL.md` from Zhixing in-memory resources.
- Close remaining `clippy::too_many_lines` warning in `xiuxian-zhixing` test lane.

## Changes

1. `xiuxian-zhixing` embedded resource export
- Added `include_dir` dependency in `packages/rust/crates/xiuxian-zhixing/Cargo.toml`.
- Exported `RESOURCES` in `packages/rust/crates/xiuxian-zhixing/src/lib.rs`:
  `pub static RESOURCES: ::include_dir::Dir<'_> = ::include_dir::include_dir!("$CARGO_MANIFEST_DIR/resources");`

2. `xiuxian-daochang` bootstrap de-legacy
- Updated `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/zhixing.rs`:
  - Removed `use xiuxian_macros::embed_utf8_dir;`
  - Removed `embed_utf8_dir!` call for built-in templates.
  - Added helper-based traversal to collect UTF-8 files from `xiuxian_zhixing::RESOURCES.get_dir("zhixing/templates")`.
- This shifts built-in template sourcing to Zhixing-owned resource image and removes macro-level duplication.

3. Zhixing resource ownership alignment
- Added built-in template files under `packages/rust/crates/xiuxian-zhixing/resources/zhixing/templates/`:
  - `daily_agenda.md`
  - `journal_reflection.md`
  - `reminder_notice.md`
  - `task_add_response.md`

4. Wendao extraction verification
- Added test in `packages/rust/crates/xiuxian-zhixing/tests/test_wendao_skill_resources.rs`:
  - `wendao_registry_extracts_skill_md_from_zhixing_resource_image`
  - Builds registry via `WendaoResourceRegistry::build_from_embedded(&RESOURCES)` and asserts `ZHIXING_SKILL_DOC_PATH` is present.

5. Long-test structural cleanup
- Refactored `packages/rust/crates/xiuxian-zhixing/tests/test_wendao_indexer.rs`:
  - Shortened `test_indexer_injects_embedded_skill_reference_graph`.
  - Extracted helper assertions:
    - `assert_skill_metadata`
    - `assert_relation_targets`
    - `assert_reference_type_hint`
- Result: removed prior `clippy::too_many_lines` warning for this function.

6. Notebook-sync test robustness update after resource-image expansion
- Updated `packages/rust/crates/xiuxian-zhixing/tests/test_heyi.rs`:
  - `test_sync_from_disk_indexes_notebook_into_wendao_graph`
  - Replaced brittle `documents.len() == 2` assertion with:
    - `documents.len() >= 2`
    - explicit presence checks for `Journal 2026-02-26` and `Agenda 2026-02-26`.
- Reason: embedded skill/reference documents are now indexed in the same graph, so exact global document count is no longer stable.

## Validation Evidence

1. Strict clippy (`xiuxian-daochang`)

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-daochang cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: no warnings/errors reported.

2. Strict clippy (`xiuxian-zhixing`)

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-zhixing cargo clippy -p xiuxian-zhixing --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: no warnings/errors reported (including prior `test_wendao_indexer.rs:186` line-count warning).

3. Targeted nextest (`xiuxian-zhixing` new integration test)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-zhixing cargo nextest run -p xiuxian-zhixing -E 'test(wendao_registry_extracts_skill_md_from_zhixing_resource_image)'
```

- Exit code: `0`
- Result: `1 passed`, `0 failed`.

4. Targeted nextest (`xiuxian-zhixing` refactored long test)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-zhixing cargo nextest run -p xiuxian-zhixing -E 'test(test_indexer_injects_embedded_skill_reference_graph)'
```

- Exit code: `0`
- Result: `1 passed`, `0 failed`.

5. Targeted nextest (`xiuxian-daochang` bootstrap bridge contract)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang -E 'test(load_skill_templates_from_embedded_registry_uses_semantic_wendao_uri_links)'
```

- Exit code: `0`
- Result: `1 passed`, `0 failed`.

6. Full crate nextest (`xiuxian-zhixing`) after robustness fix

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-zhixing cargo nextest run -p xiuxian-zhixing
```

- Exit code: `0`
- Result: `36 passed`, `0 failed`.

## Outcome

- Zhixing resources now expose a crate-level canonical in-memory image (`RESOURCES`).
- Omni bootstrap no longer depends on duplicated local `embed_utf8_dir!` usage for built-in templates.
- Wendao extraction path from the Zhixing memory image is explicitly tested.
- `xiuxian-zhixing` strict-clippy lane is warning-zero for the touched scope.
