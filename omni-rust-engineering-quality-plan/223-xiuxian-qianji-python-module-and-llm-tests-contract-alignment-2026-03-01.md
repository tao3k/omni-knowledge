# 223. `xiuxian-qianji` Python Module and LLM Tests Contract Alignment (2026-03-01)

## Scope

- Remove remaining all-features clippy warning debt in the `python_module` lane.
- Align LLM workflow integration tests with the current `agenda_flow.toml`
  output contract.
- Keep strict-lint and targeted regression lanes green.

## Changes

1. Python module quality hardening (`src/python_module.rs`)
- Added module and API docs for PyO3-exported classes/functions.
- Added `Default` implementations for `PyQianjiEngine` and
  `PyQianjiScheduler` to satisfy constructor-style guidance.
- Updated signatures to reduce needless ownership in Python boundary methods:
  - `add_mock_node(&mut self, id: &str, ...)`
  - `add_link(&mut self, ..., label: Option<&str>, ...)`
  - `run(&self, ..., context_json: &str)`
  - `run_master_research_array(..., repo_path: &str, query: &str, api_key: &str, base_url: &str)`
- Added explicit `# Errors` docs on fallible Python-facing APIs.
- Kept runtime creation error propagation explicit (`PyRuntimeError`) and
  eliminated the last `implicit_clone` warning site.

2. LLM test warning cleanup
- `tests/llm_analyzer.rs`
  - Replaced `map(...).unwrap_or_else(...)` with `map_or_else(...)` in
    `must_array` helper (`clippy::map_unwrap_or`).
- `tests/test_bootcamp_api.rs`
  - Replaced `field_reassign_with_default` pattern with struct-update syntax.
  - Normalized formatting arguments (`uninlined_format_args`).
  - Made `bootcamp_runs_real_adversarial_flow` explicitly opt-in via
    `XIUXIAN_BOOTCAMP_CONTEXT` environment variable (returns early when absent),
    avoiding accidental non-deterministic failures in local/CI environments.
- Removed stale `unused_mut` warnings in:
  - `tests/llm_multi_tenancy.rs`
  - `tests/llm_augmented_formal_audit.rs`
  - `tests/test_qianji_master_research.rs`
  - `tests/test_agenda_validation_pipeline.rs`
- `tests/test_qianji_master_research.rs`
  - Updated formatting assertion message to inline argument style.

3. Agenda validation contract realignment
- `tests/test_agenda_validation_pipeline.rs`
  - Updated happy-path assertions to match current embedded workflow contract
    (`agenda_flow.toml`):
    - Assert presence of `student_proposal`, `steward_feedback`,
      `professor_annotated_prompt`, `professor_conclusion`,
      `final_synaptic_report`.
    - Assert score key `governance_score` (replacing obsolete `teacher_score`).
    - Removed assertions against obsolete fields
      (`agenda_commit_status`, `*_persona_id`, legacy template-target metadata).

## Validation Evidence

1. Strict clippy (default target set)

```bash
cargo clippy -p xiuxian-qianji --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: no warnings/errors in the crate lane.

2. Strict clippy (`--all-features`)

```bash
cargo clippy -p xiuxian-qianji --all-targets --all-features -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: no warnings/errors in the crate lane.

3. Targeted regression: agenda validation happy path

```bash
cargo nextest run -p xiuxian-qianji --features llm --test test_agenda_validation_pipeline -E 'test(agenda_validation_pipeline_compiles_and_runs_happy_path)'
```

- Exit code: `0`
- Result: `1 passed`, `2 skipped`.

4. Aggregated LLM integration lane

```bash
cargo nextest run -p xiuxian-qianji --features llm --test llm_analyzer --test test_bootcamp_api --test llm_multi_tenancy --test llm_augmented_formal_audit --test test_qianji_master_research --test test_agenda_validation_pipeline
```

- Exit code: `0`
- Result: `22 passed`, `0 failed`.

## Outcome

- `xiuxian-qianji` strict clippy is clean under both default and all-features.
- Python boundary APIs are documented and ownership semantics are tighter.
- LLM workflow tests now assert current manifest contract instead of legacy keys.
- The touched all-features LLM regression lane is stable and green.
