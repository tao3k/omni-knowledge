# Omni Rust Engineering Quality Plan

## Purpose

This directory contains the project-specific modernization and execution plan
for `omni-dev-fusion`.

Unlike the codex reference directory, this folder is dedicated to:

- current-state diagnosis of this repository,
- prioritized implementation roadmap,
- execution checklists and tracking,
- objective verification of high-quality Rust engineering outcomes.

## Document Map

1. `00-omni-current-state-baseline.md`
   - Evidence-based baseline for current Rust/Python engineering state.
2. `01-gap-matrix-and-priorities.md`
   - Gap matrix and priority ordering.
3. `02-modern-rust-and-software-standards.md`
   - Target standards to enforce.
4. `03-zero-to-one-execution-plan.md`
   - Feature-based modernization roadmap.
5. `04-operating-checklists.md`
   - Daily/PR/CI/release execution checklists.
6. `05-evidence-metrics-snapshot-2026-02-18.md`
   - Reproducible metrics snapshot for this repository.
7. `06-high-quality-rust-engineering-scorecard.md`
   - Tracking framework to verify progress toward high-quality outcomes.
8. `07-second-pass-plan-refinement-2026-02-22.md`
   - Second-pass plan refinement aligned with latest codebase and cleanup progress.
9. `08-second-pass-revalidation-and-execution-update-2026-02-23.md`
   - Revalidated baseline and near-term execution update after latest strict-clippy cleanup progress.
10. `09-strict-clippy-progress-wave-2026-02-23.md`
   - Strict clippy convergence wave result and completion update (`xiuxian-daochang`, `xiuxian-vector`, `omni-core-rs`, and full workspace strict checks are clean with evidence logs).
11. `10-benchmark-stability-and-test-validation-2026-02-23.md`
   - Benchmark test-threshold stabilization and re-validation evidence (`xiuxian-vector` full tests + strict clippy clean).
12. `11-omni-core-rs-lib-test-runtime-note-2026-02-23.md`
   - Runtime-linking execution note for stable `omni-core-rs` test runs on macOS/Nix (`_PyBool_Type` preload pattern and validation evidence).
13. `12-rust-quality-gates-checklist-2026-02-23.md`
   - Operational Rust quality-gate sequence with reproducible commands and pass criteria.
14. `13-third-pass-codex-to-omni-adoption-plan-2026-02-23.md`
   - Third-pass project-specific adoption slices mapping Codex engineering patterns to Omni execution.
15. `14-rust-gate-automation-integration-2026-02-23.md`
   - Integration record for wiring `omni-core-rs` runtime-safe test lane into default local/CI Rust quality gates.
16. `15-rust-gate-timeout-tuning-and-script-discoverability-2026-02-23.md`
   - Follow-up record for timeout default tuning (`3600`) and script discoverability updates.
17. `16-fourth-pass-two-doc-follow-up-2026-02-23.md`
   - Fourth-pass follow-up based on the codex third-pass systems reference and the omni third-pass adoption plan, with updated slice status and next-wave execution priorities.
18. `17-dependency-security-lane-bootstrap-2026-02-24.md`
   - Dependency security lane bootstrap record: local/CI wiring, `deny.toml`
     baseline, real gate execution evidence, and staged remediation plan.
19. `18-dependency-security-exception-register-2026-02-24.md`
   - Temporary advisory exception register with owner/removal-condition tracking.
20. `19-reqwest011-transitive-decommission-plan-2026-02-24.md`
   - Execution plan to remove `reqwest 0.11`/`rustls-pemfile` transitive path
     from `xiuxian-daochang` chain.
21. `20-lru0125-transitive-elimination-plan-2026-02-24.md`
   - Execution plan to eliminate `lru 0.12.5` from `xiuxian-vector` dependency
     graph.
22. `21-xiuxian-daochang-feature-gated-profile-validation-2026-02-24.md`
   - Execution record for making `xiuxian-daochang` reduced profile
     (`--no-default-features`) compile/test green with zero warnings while
     preserving default-profile behavior.
23. `22-xiuxian-daochang-profile-matrix-ci-integration-2026-02-24.md`
   - CI enforcement record for `xiuxian-daochang` profile matrix gates in both
     `ci.yaml` and `checks.yaml`.
24. `23-xiuxian-daochang-dependency-graph-assertions-gate-2026-02-24.md`
   - CI gate record for `xiuxian-daochang` dependency-graph assertions
     (`litellm-rs` profile split + advisory transitive signals).
25. `24-xiuxian-daochang-litellm-compat-layer-2026-02-24.md`
   - Internal compatibility-layer extraction record for isolating
     `litellm-rs` integration in `xiuxian-daochang` while preserving behavior.
26. `25-rmcp016-reqwest013-xiuxian-mcp-unification-2026-02-24.md`
   - Dependency and package-boundary migration record for
     `rmcp 0.16`, `reqwest 0.13`, and `xiuxian-mcp` activation.
27. `26-tasks-nix-thin-wrapper-and-rust-env-segmentation-2026-02-24.md`
   - Task-runner boundary hardening record (`tasks.nix` thin wrapper policy)
     and Rust environment segmentation for lighter independent CI lanes.
28. `27-rust-code-quality-adoption-wave-2026-02-24.md`
   - Code-quality-focused adoption wave from Codex Rust patterns
     (typed errors, orchestration/runtime split, instrumentation, suppression debt).
29. `28-xiuxian-vector-test-quality-convergence-2026-02-26.md`
   - Strict-pedantic convergence record for `xiuxian-vector` test quality,
     including large test-module decomposition and full test/lint evidence.
30. `29-xiuxian-vector-admin-impl-modularization-2026-02-26.md`
   - Production-code modularization record for `xiuxian-vector`
     `ops/admin_impl` decomposition with full lint/test evidence.
31. `30-xiuxian-vector-writer-impl-modularization-2026-02-26.md`
   - Production-code modularization record for `xiuxian-vector`
     `ops/writer_impl` decomposition with full lint/test evidence.
32. `31-xiuxian-vector-skill-search-modularization-2026-02-26.md`
   - Production-code modularization record for `xiuxian-vector`
     `skill/ops_impl/search` decomposition with full lint/test evidence.
33. `32-xiuxian-vector-skill-scanner-lint-debt-reduction-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector` skill scanner
     (`unused_self`/`collapsible_if` root-cause fixes and full verification).
34. `33-xiuxian-vector-keyword-index-modularization-2026-02-26.md`
   - Production-code modularization record for `xiuxian-vector`
     `keyword/index` decomposition with full lint/test evidence.
35. `34-xiuxian-vector-skill-indexing-lint-debt-reduction-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     `skill/ops_impl/indexing` (`doc_markdown`/`unused_self` root-cause fixes).
36. `35-xiuxian-vector-search-impl-modularization-2026-02-26.md`
   - Production-code modularization record for `xiuxian-vector`
     `search/search_impl/mod` decomposition with full lint/test evidence.
37. `36-xiuxian-vector-admin-guards-lint-debt-reduction-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     `ops/admin_impl/guards` root-cause cleanup and call-site updates.
38. `37-xiuxian-vector-admin-impl-doc-markdown-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     `ops/admin_impl` doc-markdown cleanup across table/delete/index modules.
39. `38-xiuxian-vector-skill-ops-doc-markdown-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     `skill/ops_impl` doc-markdown cleanup across listing/registry modules.
40. `39-xiuxian-vector-skill-mod-doc-markdown-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     `skill/mod` module-entry doc-markdown cleanup.
41. `40-xiuxian-vector-search-cast-truncation-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector` search layer
     cast-truncation cleanup (`f64 -> f32` safe conversion path).
42. `41-xiuxian-vector-pass-by-value-suppression-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     pass-by-value cleanup in keyword fusion and writer ingest paths.
43. `42-xiuxian-vector-entity-aware-pass-by-value-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     entity-aware pass-by-value cleanup.
44. `43-xiuxian-vector-checkpoint-collapsible-if-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     checkpoint store collapsible-if cleanup.
45. `44-xiuxian-vector-ops-collapsible-if-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     additional ops/checkpoint collapsible-if cleanup wave.
46. `45-xiuxian-vector-core-constructor-collapsible-if-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     core constructor collapsible-if cleanup.
47. `46-xiuxian-vector-writer-impl-suppression-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     writer implementation suppression cleanup wave.
48. `47-xiuxian-vector-doc-markdown-and-unused-async-convergence-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-vector`
     doc-markdown cleanup convergence and `unused_async` root-cause fixes.
49. `48-omni-core-rs-pedantic-cleanup-wave-2026-02-26.md`
   - Lint-suppression debt reduction record for `omni-core-rs`
     pedantic cleanup across skill-index tests and AST/tag test modules.
50. `49-xiuxian-daochang-pedantic-test-cleanup-wave-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-daochang`
     pedantic cleanup across config and native-tool test lanes.
51. `50-omni-memory-pedantic-cleanup-wave-2026-02-26.md`
   - Lint-suppression debt reduction record for `omni-memory`
     pedantic cleanup and full test-lane revalidation.
52. `51-xiuxian-zhixing-reminder-queue-test-suppression-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-zhixing`
     reminder-queue test-lane suppression cleanup.
53. `52-xiuxian-wendao-storage-tests-expect-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-wendao`
     storage unit-test expect-style cleanup.
54. `53-omni-memory-cast-precision-convergence-2026-02-26.md`
   - Lint-suppression debt reduction record for `omni-memory`
     cast-precision cleanup convergence in production sources.
55. `54-xiuxian-wendao-sync-tests-suppression-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-wendao`
     sync unit-test suppression cleanup.
56. `55-xiuxian-wendao-dependency-indexer-test-suppression-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-wendao`
     dependency-indexer test suppression cleanup.
57. `56-xiuxian-wendao-storage-true-async-valkey-convergence-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-wendao`
     storage true-async Valkey convergence.
58. `57-xiuxian-wendao-doc-markdown-cleanup-wave-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-wendao`
     doc-markdown cleanup wave.
59. `58-xiuxian-wendao-allow-debt-reduction-wave-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-wendao`
     follow-up allow-debt reduction (`needless_pass_by_value`/`unused_self`).
60. `59-xiuxian-wendao-graph-api-and-py-bridge-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-wendao`
     graph API and Python-bridge cleanup (`unnecessary_wraps` + boundary fixes).
61. `60-xiuxian-wendao-convergence-to-two-suppressions-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-wendao`
     convergence pass reducing suppression debt to two domain-model entries.
62. `61-xiuxian-wendao-zero-suppression-convergence-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-wendao`
     final convergence to zero suppression attributes in `src`.
63. `62-xiuxian-daochang-native-tools-and-config-test-convergence-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-daochang`
     config/bootstrap/native-tools convergence plus `llm_proxy` expect cleanup.
64. `63-xiuxian-daochang-unused-self-suppression-cleanup-2026-02-26.md`
   - Lint-suppression debt reduction record for `xiuxian-daochang`
     associated-function cleanup and zero `unused_self` suppressions.
65. `64-xiuxian-zhixing-heyi-pedantic-warning-reduction-2026-02-26.md`
   - Pedantic warning reduction record for `xiuxian-zhixing` `heyi` lane and
     Wendao indexer test `expect()` cleanup.
66. `65-xiuxian-daochang-ref-option-and-runtime-startup-cleanup-2026-02-26.md`
   - Suppression-debt reduction record for `xiuxian-daochang` `ref_option` removal
     and Telegram runtime startup signature cleanup.
67. `66-xiuxian-daochang-type-complexity-and-unnecessary-wraps-convergence-2026-02-26.md`
   - Suppression-debt reduction record for `xiuxian-daochang` convergence on
     `type_complexity`/`unnecessary_wraps` and startup-boundary validation.
68. `67-xiuxian-daochang-session-budget-pass-by-value-cleanup-2026-02-26.md`
   - Suppression-debt reduction record for `xiuxian-daochang` session-budget
     formatter borrow conversion and `large_types_passed_by_value` elimination.
69. `68-xiuxian-daochang-wildcard-import-and-channel-runner-cleanup-2026-02-26.md`
   - Suppression-debt reduction record for `xiuxian-daochang` wildcard-import
     elimination and Telegram channel-runner interface cleanup.
70. `69-xiuxian-daochang-cast-conversion-convergence-2026-02-26.md`
   - Suppression-debt reduction record for `xiuxian-daochang` cast-conversion
     convergence and elimination of cast-related suppression categories.
71. `70-xiuxian-daochang-struct-field-names-compat-migration-plan-2026-02-26.md`
   - Design-only compatibility migration plan for eliminating the final
     `struct_field_names` suppression category in `xiuxian-daochang`.
72. `71-xiuxian-daochang-struct-field-names-delegacy-convergence-2026-02-26.md`
   - Implementation and evidence record for direct (non-legacy) field-name
     convergence and zero `struct_field_names` suppressions in
     `xiuxian-daochang/src`.
73. `72-xiuxian-qianhuan-and-zhixing-pedantic-convergence-wave-2026-02-26.md`
   - Pedantic convergence record for `xiuxian-qianhuan` and
     `xiuxian-zhixing`, including no-suppression production fixes and
     `expect`-free test migration evidence.
74. `73-xiuxian-mcp-and-llm-test-pedantic-convergence-wave-2026-02-26.md`
   - Pedantic convergence record for `xiuxian-mcp` and `xiuxian-llm` test
     lanes, including `expect`/`unwrap` removal and strict-gate evidence.
75. `74-xiuxian-llm-test-allow-debt-reduction-wave-2026-02-26.md`
   - Follow-up `xiuxian-llm` test-lane convergence record reducing file-level
     `allow` debt (`expect_used`/`unwrap_used`) in MCP runtime tests.
76. `75-xiuxian-llm-test-expect-unwrap-zero-convergence-2026-02-26.md`
   - Final convergence record for removing all remaining
     `expect_used`/`unwrap_used` suppressions in `xiuxian-llm` test lanes,
     with `Result`-first runtime test setup and full pedantic verification.
77. `76-xiuxian-daochang-test-allow-debt-and-hot-reload-isolation-wave-2026-02-26.md`
   - Focused `xiuxian-daochang` test-lane convergence record: allow-debt reduction,
     hot-reload clippy fixes, and child-process isolation to prevent env
     leakage across concurrent tests.
78. `77-xiuxian-daochang-telegram-media-test-allow-debt-reduction-wave-2026-02-26.md`
   - Telegram media test-lane convergence record in `xiuxian-daochang`: removal of
     stale `expect/unwrap` suppressions plus root-cause replacement of an
     exposed `expect()` path.
79. `78-xiuxian-daochang-webhook-and-runtime-test-allow-debt-reduction-wave-2026-02-26.md`
   - Webhook/runtime test-lane convergence record in `xiuxian-daochang`: additional
     stale suppression cleanup plus root-cause replacement of remaining
     `expect()` receive paths in concurrent webhook tests.
80. `79-xiuxian-daochang-channels-telegram-expect-cleanup-wave-2026-02-26.md`
   - Focused convergence record for `xiuxian-daochang` `channels_telegram` test
     target: removal of stale `expect/unwrap` suppressions and full
     replacement of `expect`/`expect_err` paths.
81. `80-xiuxian-daochang-gateway-and-channel-test-allow-debt-reduction-wave-2026-02-26.md`
   - Gateway/channel integration convergence record in `xiuxian-daochang`: batch
     removal of stale `expect/unwrap` suppressions across managed-command,
     session-partition, telegram-markdown, gateway, and MCP startup tests.
82. `81-xiuxian-daochang-embedding-test-allow-debt-reduction-wave-2026-02-26.md`
   - Embedding-lane convergence record in `xiuxian-daochang`: suppression cleanup in
     backend/transport test modules and perf-smoke entrypoint.
83. `82-xiuxian-daochang-telegram-discord-runtime-test-allow-debt-reduction-wave-2026-02-26.md`
   - Telegram/Discord runtime test convergence record in `xiuxian-daochang`: batch
     removal of stale `expect/unwrap` suppressions and source-level pedantic
     warning fix in bootstrap tests.
84. `83-xiuxian-daochang-runtime-media-and-agent-test-allow-debt-reduction-wave-2026-02-26.md`
   - Additional convergence record in `xiuxian-daochang`: runtime/media/agent test
     suppression cleanup wave plus hot-reload assertion sync to current
     incremental mode detail.
85. `84-xiuxian-daochang-single-occurrence-expect-cleanup-wave-2026-02-26.md`
   - Focused convergence record for the next `xiuxian-daochang` single-occurrence
     `expect/unwrap` batch with root-cause replacements and marker reduction
     from `56` to `46`.
86. `85-xiuxian-daochang-two-occurrence-expect-cleanup-wave-2026-02-26.md`
   - Follow-up convergence record for the next `xiuxian-daochang` two-occurrence
     batch with root-cause replacements and marker reduction from `46` to `35`.
87. `86-xiuxian-daochang-three-occurrence-expect-cleanup-wave-2026-02-26.md`
   - Follow-up convergence record for the next `xiuxian-daochang` three-occurrence
     batch with root-cause replacements and marker reduction from `35` to `31`.
88. `87-xiuxian-daochang-four-occurrence-expect-cleanup-wave-2026-02-26.md`
   - Follow-up convergence record for the next `xiuxian-daochang` four-occurrence
     batch with root-cause replacements and marker reduction from `31` to `25`.
89. `88-xiuxian-daochang-five-occurrence-expect-cleanup-wave-2026-02-26.md`
   - Follow-up convergence record for the next `xiuxian-daochang` five-occurrence
     batch with root-cause replacements and marker reduction from `25` to `21`.
90. `89-xiuxian-daochang-six-occurrence-expect-cleanup-wave-2026-02-26.md`
   - Follow-up convergence record for the next `xiuxian-daochang` six-occurrence
     batch with root-cause replacements and marker reduction from `21` to `19`.
91. `90-xiuxian-daochang-seven-occurrence-expect-cleanup-wave-2026-02-26.md`
   - Follow-up convergence record for the next `xiuxian-daochang` seven-occurrence
     batch with root-cause replacements and marker reduction from `19` to `16`.
92. `91-xiuxian-daochang-eight-occurrence-expect-cleanup-wave-2026-02-26.md`
   - Follow-up convergence record for the next `xiuxian-daochang` eight-occurrence
     batch with root-cause replacements and marker reduction from `16` to `13`.
93. `92-xiuxian-daochang-config-and-jobs-expect-cleanup-wave-2026-02-26.md`
   - Focused convergence record for `config_mcp` and `jobs_manager` test lanes
     with root-cause replacements and marker reduction from `13` to `11`.
94. `93-xiuxian-daochang-gateway-http-expect-cleanup-wave-2026-02-26.md`
   - Focused convergence record for `gateway_http` test lane with root-cause
     replacements and marker reduction from `11` to `10`.
95. `94-xiuxian-daochang-session-gate-and-graph-executor-expect-cleanup-wave-2026-02-26.md`
   - Focused convergence record for `telegram_session_gate` and
     `agent/graph_executor` test lanes with marker reduction from `10` to `8`.
96. `95-xiuxian-daochang-discord-parsing-and-telegram-markdown-cleanup-wave-2026-02-26.md`
   - Focused convergence record for `channels_discord_parsing` and
     `channels_telegram_markdown` cleanup with marker reduction from `8` to
     `7`.
97. `96-xiuxian-daochang-embedding-client-expect-cleanup-wave-2026-02-26.md`
   - Focused convergence record for `embedding_client` cleanup with marker
     reduction from `7` to `6`.
98. `97-xiuxian-daochang-config-and-command-parsers-expect-cleanup-wave-2026-02-26.md`
   - Focused convergence record for `config_settings` and `channels_commands`
     cleanup with marker reduction from `6` to `4`.
99. `98-xiuxian-daochang-memory-test-lanes-expect-cleanup-wave-2026-02-26.md`
   - Focused convergence record for memory gate/persistence backend test-lane
     cleanup with marker reduction from `4` to `2`.
100. `99-xiuxian-daochang-final-expect-unwrap-zero-convergence-2026-02-26.md`
   - Final convergence record for zero marker files in `xiuxian-daochang/tests`
     (`expect_used`/`unwrap_used`).

## How To Use

1. Start from `06-high-quality-rust-engineering-scorecard.md` and
   `00-omni-current-state-baseline.md`.
2. Execute features in `03-zero-to-one-execution-plan.md` using canonical
   feature names.
3. Apply the updated sequencing and quality gates in
   `07-second-pass-plan-refinement-2026-02-22.md`.
4. Apply latest revalidation and near-term actions in
   `08-second-pass-revalidation-and-execution-update-2026-02-23.md`.
5. Execute routine checks using
   `12-rust-quality-gates-checklist-2026-02-23.md`.
6. Apply third-pass adoption slices in
   `13-third-pass-codex-to-omni-adoption-plan-2026-02-23.md`.
7. For each feature update, attach evidence (PR links, commands, test results)
   to the scorecard and snapshot files.
8. If workspace compile-check runs are slow on local hosts, run
   `just rust-check 3600` (or higher) before the full `just rust-quality-gate`
   sequence.
9. Use `16-fourth-pass-two-doc-follow-up-2026-02-23.md` as the active
   progress tracker for the current codex-aligned modernization wave.
10. Use `17-dependency-security-lane-bootstrap-2026-02-24.md` as the
    execution reference for the current dependency-security cleanup cycle.
11. Use `18-dependency-security-exception-register-2026-02-24.md` to
    manage temporary advisory exceptions and removal accountability.
12. Execute dependency-chain elimination steps using:
    `19-reqwest011-transitive-decommission-plan-2026-02-24.md` and
    `20-lru0125-transitive-elimination-plan-2026-02-24.md`.
13. Use `21-xiuxian-daochang-feature-gated-profile-validation-2026-02-24.md`
    as the latest evidence baseline for profile-split compile/test gates in
    `xiuxian-daochang`.
14. Use `22-xiuxian-daochang-profile-matrix-ci-integration-2026-02-24.md`
    for CI policy tracking of profile-split enforcement.
15. Use `23-xiuxian-daochang-dependency-graph-assertions-gate-2026-02-24.md`
    for dependency-graph regression checks and transitive cleanup telemetry.
16. Use `24-xiuxian-daochang-litellm-compat-layer-2026-02-24.md` for the current
    `xiuxian-daochang` adapter-boundary baseline before attempting broader dependency
    convergence.
17. Use `25-rmcp016-reqwest013-xiuxian-mcp-unification-2026-02-24.md` as the
    current baseline for MCP lane upgrades and package-boundary unification.
18. Use `26-tasks-nix-thin-wrapper-and-rust-env-segmentation-2026-02-24.md`
    for current `tasks.nix`/`just` boundary policy and Rust lane dependency
    segmentation baselines.
19. Use `27-rust-code-quality-adoption-wave-2026-02-24.md` as the active
    code-quality modernization tracker for non-CI Rust engineering upgrades.
20. Use `28-xiuxian-vector-test-quality-convergence-2026-02-26.md` for the
    latest strict-pedantic `xiuxian-vector` test quality baseline and execution
    evidence.
21. Use `29-xiuxian-vector-admin-impl-modularization-2026-02-26.md` for the
    latest `xiuxian-vector` production admin-layer modularization baseline.
22. Use `30-xiuxian-vector-writer-impl-modularization-2026-02-26.md` for the
    latest `xiuxian-vector` production writer-layer modularization baseline.
23. Use `31-xiuxian-vector-skill-search-modularization-2026-02-26.md` for the
    latest `xiuxian-vector` skill search-layer modularization baseline.
24. Use `32-xiuxian-vector-skill-scanner-lint-debt-reduction-2026-02-26.md` for
    the latest `xiuxian-vector` skill scanner suppression-debt reduction baseline.
25. Use `33-xiuxian-vector-keyword-index-modularization-2026-02-26.md` for the
    latest `xiuxian-vector` keyword-index modularization baseline.
26. Use `34-xiuxian-vector-skill-indexing-lint-debt-reduction-2026-02-26.md` for
    the latest `xiuxian-vector` skill-indexing suppression-debt reduction baseline.
27. Use `35-xiuxian-vector-search-impl-modularization-2026-02-26.md` for the
    latest `xiuxian-vector` search implementation modularization baseline.
28. Use `36-xiuxian-vector-admin-guards-lint-debt-reduction-2026-02-26.md` for
    the latest `xiuxian-vector` admin-guard suppression-debt reduction baseline.
29. Use `37-xiuxian-vector-admin-impl-doc-markdown-cleanup-2026-02-26.md` for
    the latest `xiuxian-vector` admin-impl doc-markdown cleanup baseline.
30. Use `38-xiuxian-vector-skill-ops-doc-markdown-cleanup-2026-02-26.md` for the
    latest `xiuxian-vector` skill-ops doc-markdown cleanup baseline.
31. Use `39-xiuxian-vector-skill-mod-doc-markdown-cleanup-2026-02-26.md` for the
    latest `xiuxian-vector` skill-module doc-markdown cleanup baseline.
32. Use `40-xiuxian-vector-search-cast-truncation-cleanup-2026-02-26.md` for the
    latest `xiuxian-vector` search-layer cast-truncation cleanup baseline.
33. Use `41-xiuxian-vector-pass-by-value-suppression-cleanup-2026-02-26.md` for
    the latest `xiuxian-vector` pass-by-value suppression cleanup baseline.
34. Use `42-xiuxian-vector-entity-aware-pass-by-value-cleanup-2026-02-26.md` for
    the latest `xiuxian-vector` entity-aware pass-by-value cleanup baseline.
35. Use `43-xiuxian-vector-checkpoint-collapsible-if-cleanup-2026-02-26.md` for
    the latest `xiuxian-vector` checkpoint-store collapsible-if cleanup baseline.
36. Use `44-xiuxian-vector-ops-collapsible-if-cleanup-2026-02-26.md` for the
    latest `xiuxian-vector` ops/checkpoint collapsible-if cleanup baseline.
37. Use `45-xiuxian-vector-core-constructor-collapsible-if-cleanup-2026-02-26.md`
    for the latest `xiuxian-vector` core-constructor collapsible-if cleanup
    baseline.
38. Use `46-xiuxian-vector-writer-impl-suppression-cleanup-2026-02-26.md` for the
    latest `xiuxian-vector` writer-implementation suppression cleanup baseline.
39. Use `47-xiuxian-vector-doc-markdown-and-unused-async-convergence-2026-02-26.md`
    for the latest `xiuxian-vector` doc-markdown and `unused_async`
    convergence baseline.
40. Use `48-omni-core-rs-pedantic-cleanup-wave-2026-02-26.md` for the latest
    `omni-core-rs` pedantic lint-cleanup baseline and verification evidence.
41. Use `49-xiuxian-daochang-pedantic-test-cleanup-wave-2026-02-26.md` for the latest
    `xiuxian-daochang` pedantic test-lane cleanup baseline and verification evidence.
42. Use `50-omni-memory-pedantic-cleanup-wave-2026-02-26.md` for the latest
    `omni-memory` pedantic cleanup baseline and full test-lane verification.
43. Use `51-xiuxian-zhixing-reminder-queue-test-suppression-cleanup-2026-02-26.md`
    for the latest `xiuxian-zhixing` reminder-queue test-lane cleanup
    baseline.
44. Use `52-xiuxian-wendao-storage-tests-expect-cleanup-2026-02-26.md` for the
    latest `xiuxian-wendao` storage unit-test cleanup baseline.
45. Use `53-omni-memory-cast-precision-convergence-2026-02-26.md` for the
    latest `omni-memory` cast-precision convergence baseline.
46. Use `54-xiuxian-wendao-sync-tests-suppression-cleanup-2026-02-26.md` for
    the latest `xiuxian-wendao` sync unit-test cleanup baseline.
47. Use `55-xiuxian-wendao-dependency-indexer-test-suppression-cleanup-2026-02-26.md`
    for the latest `xiuxian-wendao` dependency-indexer test cleanup baseline.
48. Use `56-xiuxian-wendao-storage-true-async-valkey-convergence-2026-02-26.md`
    for the latest `xiuxian-wendao` storage true-async convergence baseline.
49. Use `57-xiuxian-wendao-doc-markdown-cleanup-wave-2026-02-26.md` for the
    latest `xiuxian-wendao` doc-markdown cleanup baseline.
50. Use `58-xiuxian-wendao-allow-debt-reduction-wave-2026-02-26.md` for the
    latest `xiuxian-wendao` allow-debt reduction baseline.
51. Use `59-xiuxian-wendao-graph-api-and-py-bridge-cleanup-2026-02-26.md` for
    the latest `xiuxian-wendao` graph API and Python-bridge cleanup baseline.
52. Use `60-xiuxian-wendao-convergence-to-two-suppressions-2026-02-26.md` for
    the latest `xiuxian-wendao` suppression convergence baseline.
53. Use `61-xiuxian-wendao-zero-suppression-convergence-2026-02-26.md` for the
    latest `xiuxian-wendao` zero-suppression convergence baseline.
54. Use `62-xiuxian-daochang-native-tools-and-config-test-convergence-2026-02-26.md`
    for the latest `xiuxian-daochang` config/bootstrap/native-tools convergence
    baseline and `llm_proxy` expect-cleanup evidence.
55. Use `63-xiuxian-daochang-unused-self-suppression-cleanup-2026-02-26.md` for the
    latest `xiuxian-daochang` associated-function convergence and `unused_self`
    suppression cleanup evidence.
56. Use `64-xiuxian-zhixing-heyi-pedantic-warning-reduction-2026-02-26.md` for
    the latest `xiuxian-zhixing` `heyi` pedantic warning reduction baseline.
57. Use `65-xiuxian-daochang-ref-option-and-runtime-startup-cleanup-2026-02-26.md`
    for the latest `xiuxian-daochang` `ref_option` and runtime startup cleanup
    baseline.
58. Use `71-xiuxian-daochang-struct-field-names-delegacy-convergence-2026-02-26.md`
    for the latest no-legacy convergence evidence on `McpSettings` and
    Telegram/Discord slash-policy naming in `xiuxian-daochang`.
59. Use `72-xiuxian-qianhuan-and-zhixing-pedantic-convergence-wave-2026-02-26.md`
    for the latest cross-crate pedantic convergence baseline and `expect`-free
    qianhuan test migration evidence.
60. Use `73-xiuxian-mcp-and-llm-test-pedantic-convergence-wave-2026-02-26.md`
    for the latest strict pedantic convergence baseline in MCP/LLM integration
    test lanes.
61. Use `74-xiuxian-llm-test-allow-debt-reduction-wave-2026-02-26.md` for the
    latest `xiuxian-llm` MCP runtime test allow-debt reduction evidence.
62. Use `75-xiuxian-llm-test-expect-unwrap-zero-convergence-2026-02-26.md`
    for the latest zero-occurrence convergence evidence of
    `expect_used`/`unwrap_used` suppressions across `xiuxian-llm` tests.
63. Use `76-xiuxian-daochang-test-allow-debt-and-hot-reload-isolation-wave-2026-02-26.md`
    for the latest `xiuxian-daochang` test-lane allow-debt reduction and hot-reload
    isolation hardening evidence.
64. Use `77-xiuxian-daochang-telegram-media-test-allow-debt-reduction-wave-2026-02-26.md`
    for the latest Telegram media test-lane suppression-debt reduction and
    panic-path cleanup evidence in `xiuxian-daochang`.
65. Use `78-xiuxian-daochang-webhook-and-runtime-test-allow-debt-reduction-wave-2026-02-26.md`
    for the latest webhook/runtime suppression-debt reduction and concurrent
    receive-path panic cleanup evidence in `xiuxian-daochang`.
66. Use `79-xiuxian-daochang-channels-telegram-expect-cleanup-wave-2026-02-26.md`
    for the latest `channels_telegram` test-lane expect-cleanup convergence in
    `xiuxian-daochang`.
67. Use `80-xiuxian-daochang-gateway-and-channel-test-allow-debt-reduction-wave-2026-02-26.md`
    for the latest gateway/channel batch suppression cleanup evidence in
    `xiuxian-daochang`.
68. Use `81-xiuxian-daochang-embedding-test-allow-debt-reduction-wave-2026-02-26.md`
    for the latest embedding-lane suppression cleanup evidence in `xiuxian-daochang`.
69. Use `82-xiuxian-daochang-telegram-discord-runtime-test-allow-debt-reduction-wave-2026-02-26.md`
    for the latest Telegram/Discord runtime suppression cleanup evidence in
    `xiuxian-daochang`.
70. Use `83-xiuxian-daochang-runtime-media-and-agent-test-allow-debt-reduction-wave-2026-02-26.md`
    for the latest runtime/media/agent suppression cleanup evidence in
    `xiuxian-daochang`.
71. Use `84-xiuxian-daochang-single-occurrence-expect-cleanup-wave-2026-02-26.md`
    for the latest single-occurrence `expect/unwrap` convergence evidence and
    updated next-queue baseline in `xiuxian-daochang` tests.
72. Use `85-xiuxian-daochang-two-occurrence-expect-cleanup-wave-2026-02-26.md` for
    the latest two-occurrence `expect/unwrap` convergence evidence and updated
    three-occurrence next queue in `xiuxian-daochang` tests.
73. Use `86-xiuxian-daochang-three-occurrence-expect-cleanup-wave-2026-02-26.md` for
    the latest three-occurrence `expect/unwrap` convergence evidence and
    updated four-occurrence next queue in `xiuxian-daochang` tests.
74. Use `87-xiuxian-daochang-four-occurrence-expect-cleanup-wave-2026-02-26.md` for
    the latest four-occurrence `expect/unwrap` convergence evidence and
    updated five-occurrence next queue in `xiuxian-daochang` tests.
75. Use `88-xiuxian-daochang-five-occurrence-expect-cleanup-wave-2026-02-26.md` for
    the latest five-occurrence `expect/unwrap` convergence evidence and
    updated six-occurrence next queue in `xiuxian-daochang` tests.
76. Use `89-xiuxian-daochang-six-occurrence-expect-cleanup-wave-2026-02-26.md` for
    the latest six-occurrence `expect/unwrap` convergence evidence and updated
    seven-occurrence next queue in `xiuxian-daochang` tests.
77. Use `90-xiuxian-daochang-seven-occurrence-expect-cleanup-wave-2026-02-26.md` for
    the latest seven-occurrence `expect/unwrap` convergence evidence and
    updated eight-occurrence next queue in `xiuxian-daochang` tests.
78. Use `91-xiuxian-daochang-eight-occurrence-expect-cleanup-wave-2026-02-26.md` for
    the latest eight-occurrence `expect/unwrap` convergence evidence and
    updated post-wave baseline in `xiuxian-daochang` tests.
79. Use `92-xiuxian-daochang-config-and-jobs-expect-cleanup-wave-2026-02-26.md` for
    the latest `config_mcp` and `jobs_manager` expect-cleanup evidence in
    `xiuxian-daochang` tests.
80. Use `93-xiuxian-daochang-gateway-http-expect-cleanup-wave-2026-02-26.md` for
    the latest `gateway_http` expect-cleanup evidence and current residual
    marker baseline in `xiuxian-daochang` tests.
81. Use `94-xiuxian-daochang-session-gate-and-graph-executor-expect-cleanup-wave-2026-02-26.md`
    for the latest `telegram_session_gate` and `agent/graph_executor`
    expect-cleanup convergence evidence in `xiuxian-daochang` tests.
82. Use `95-xiuxian-daochang-discord-parsing-and-telegram-markdown-cleanup-wave-2026-02-26.md`
    for the latest parsing/rendering lane suppression cleanup evidence in
    `xiuxian-daochang` tests.
83. Use `96-xiuxian-daochang-embedding-client-expect-cleanup-wave-2026-02-26.md` for
    the latest embedding-client suppression cleanup evidence and current
    residual marker baseline in `xiuxian-daochang` tests.
84. Use `97-xiuxian-daochang-config-and-command-parsers-expect-cleanup-wave-2026-02-26.md`
    for the latest config/command parser suppression cleanup evidence in
    `xiuxian-daochang` tests.
85. Use `98-xiuxian-daochang-memory-test-lanes-expect-cleanup-wave-2026-02-26.md`
    for the latest memory gate/persistence suppression cleanup evidence in
    `xiuxian-daochang` tests.
86. Use `99-xiuxian-daochang-final-expect-unwrap-zero-convergence-2026-02-26.md`
    for the final zero-marker convergence baseline in `xiuxian-daochang/tests`.
87. Use `100-incremental-sync-policy-api-alignment-and-cross-crate-validation-2026-02-26.md`
    for second-pass cross-crate API alignment verification of
    `IncrementalSyncPolicy::new(&[String])` and preserved `xiuxian-daochang`
    quality baselines.
88. Use `101-xiuxian-qianji-test-lane-expect-unwrap-pedantic-convergence-2026-02-26.md`
    for second-pass `xiuxian-qianji` test-lane `expect/unwrap` cleanup and
    pedantic convergence evidence.
89. Use `102-xiuxian-qianji-test-marker-zero-convergence-2026-02-26.md` for
    second-pass file-level marker zero convergence in `xiuxian-qianji/tests`.
90. Use `103-xiuxian-wendao-test-marker-reduction-wave-2026-02-26.md` for
    second-pass marker reduction and pedantic verification evidence in
    `xiuxian-wendao/tests`.
91. Use `104-xiuxian-wendao-sync-and-query-marker-reduction-wave-2026-02-26.md`
    for the sync/query low-occurrence marker reduction continuation in
    `xiuxian-wendao/tests`.
92. Use `105-xiuxian-wendao-test-marker-zero-convergence-2026-02-26.md` for
    final file-level marker zero convergence evidence in
    `xiuxian-wendao/tests`.
93. Use `106-rust-tests-global-marker-zero-snapshot-2026-02-26.md` for the
    workspace-level marker-zero snapshot across `packages/rust/crates/**/tests`.
94. Use `107-xiuxian-wendao-map-unwrap-or-convergence-and-enhancer-test-repair-2026-02-26.md`
    for post-marker `map_unwrap_or` convergence and enhancer test import repair
    evidence in `xiuxian-wendao`.
95. Use `108-xiuxian-daochang-map-unwrap-or-zero-and-clippy-revalidation-2026-02-27.md`
    for `xiuxian-daochang` test-lane `map_unwrap_or` suppression elimination,
    source-level warning fixes, and strict clippy revalidation evidence.
96. Use `109-xiuxian-daochang-redundant-closure-convergence-2026-02-27.md` for
    `xiuxian-daochang` test-lane `redundant_closure_for_method_calls` suppression
    elimination and post-fix strict clippy convergence evidence.
97. Use `110-xiuxian-daochang-option-deref-and-raw-string-convergence-2026-02-27.md`
    for `xiuxian-daochang` convergence on `option_as_ref_deref` and
    `needless_raw_string_hashes`, plus `config_settings` marker removal for
    `expect_used`/`unwrap_used`.
98. Use `111-xiuxian-daochang-useless-conversion-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::useless_conversion` with strict
    pedantic/too-many-lines revalidation evidence.
99. Use `112-xiuxian-daochang-unreadable-literal-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::unreadable_literal` with strict
    pedantic/too-many-lines revalidation evidence.
100. Use `113-xiuxian-daochang-manual-string-new-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::manual_string_new` with strict
    pedantic/too-many-lines revalidation evidence.
101. Use `114-xiuxian-daochang-manual-let-else-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::manual_let_else` with strict
    pedantic/too-many-lines revalidation evidence.
102. Use `115-xiuxian-daochang-manual-assert-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::manual_assert` with strict
    pedantic/too-many-lines revalidation evidence.
103. Use `116-xiuxian-daochang-format-collect-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::format_collect` with strict
    pedantic/too-many-lines revalidation evidence.
104. Use `117-xiuxian-daochang-async-yields-async-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::async_yields_async` with strict
    pedantic/too-many-lines revalidation evidence.
105. Use `118-xiuxian-daochang-assigning-clones-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::assigning_clones` with strict
    pedantic/too-many-lines revalidation evidence.
106. Use `119-xiuxian-daochang-single-match-else-marker-zero-and-zhenfa-followup-2026-02-27.md`
    for `xiuxian-daochang` `clippy::single_match_else` marker-zero convergence and
    follow-up fixes in `agent/mcp` and `agent/zhenfa` paths.
107. Use `120-xiuxian-daochang-manual-async-fn-convergence-and-cross-crate-clean-pass-2026-02-27.md`
    for `xiuxian-daochang` `clippy::manual_async_fn` convergence and cross-crate
    strict-clippy revalidation (`xiuxian-wendao` + `xiuxian-daochang`).
108. Use `121-xiuxian-daochang-match-wildcard-single-variants-convergence-2026-02-27.md`
    for `xiuxian-daochang` convergence on
    `clippy::match_wildcard_for_single_variants` with strict revalidation
    evidence.
109. Use `122-xiuxian-daochang-unnecessary-literal-bound-convergence-2026-02-27.md`
    for `xiuxian-daochang` convergence on `clippy::unnecessary_literal_bound`,
    including `'static` `name()` signature alignment in runtime mock channels.
110. Use `123-xiuxian-daochang-similar-names-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::similar_names` with explicit
    variable-name disambiguation evidence.
111. Use `124-xiuxian-daochang-clippy-panic-marker-zero-convergence-2026-02-27.md`
    for `xiuxian-daochang` convergence to zero `clippy::panic` marker usage with
    strict revalidation evidence.
112. Use `125-xiuxian-daochang-missing-panics-doc-marker-zero-convergence-2026-02-27.md`
    for `xiuxian-daochang` convergence to zero `clippy::missing_panics_doc` marker
    usage with strict revalidation evidence.
113. Use `126-xiuxian-daochang-cast-lossless-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::cast_lossless` with strict
    revalidation evidence.
114. Use `127-xiuxian-daochang-cast-possible-wrap-and-zhenfa-followup-convergence-2026-02-27.md`
    for `xiuxian-daochang` convergence on `clippy::cast_possible_wrap` and follow-up
    zhenfa pedantic warning resolution.
115. Use `128-xiuxian-daochang-uninlined-format-args-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::uninlined_format_args` with strict
    pedantic/too-many-lines revalidation evidence.
116. Use `129-xiuxian-daochang-needless-pass-by-value-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::needless_pass_by_value` with
    signature-by-reference fixes in discord runtime test support paths.
117. Use `130-xiuxian-daochang-doc-markdown-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::doc_markdown` with strict
    revalidation evidence across `xiuxian-daochang` and `xiuxian-zhenfa`.
118. Use `131-xiuxian-daochang-cast-sign-loss-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::cast_sign_loss` with safe percentile
    index conversion refactors in test lanes.
119. Use `132-xiuxian-daochang-cast-possible-truncation-convergence-2026-02-27.md`
    for `xiuxian-daochang` convergence on `clippy::cast_possible_truncation` with
    checked epoch-millisecond conversion in `session_redis` tests.
120. Use `133-xiuxian-daochang-cast-precision-loss-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::cast_precision_loss` with integer
    thresholding and checked narrowing in embedding/reflection test lanes.
121. Use `134-xiuxian-daochang-field-reassign-with-default-convergence-2026-02-27.md`
    for `xiuxian-daochang` convergence on `clippy::field_reassign_with_default` and
    cross-crate `clippy::unnecessary_literal_bound` spillover resolution.
122. Use `135-xiuxian-daochang-float-cmp-convergence-2026-02-27.md` for
    `xiuxian-daochang` convergence on `clippy::float_cmp` with tolerance-based
    assertions in memory/config/discover cache test lanes.
123. Use `136-xiuxian-daochang-struct-field-and-large-futures-convergence-2026-02-27.md`
    for `xiuxian-daochang` post-`struct_field_names` follow-up convergence
    (`manual_let_else` + `large_futures`) and cross-crate wendao pedantic
    warning resolution with full strict-clippy evidence.
124. Use `137-xiuxian-daochang-small-test-marker-burndown-and-docs-replacement-2026-02-27.md`
    for incremental `xiuxian-daochang/tests` marker-debt reduction in small modules
    (`too_many_lines`/`too_many_arguments`) and `missing_docs` replacement with
    explicit test documentation.
125. Use `138-xiuxian-daochang-marker-burndown-wave-2-and-cross-crate-pedantic-cleanup-2026-02-27.md`
    for the second marker-burndown wave in `xiuxian-daochang/tests` plus cross-crate
    pedantic cleanup in `xiuxian-daochang` bootstrap and `xiuxian-wendao` XML-Lite
    path typing.
126. Use `139-xiuxian-daochang-session-preemption-marker-convergence-2026-02-27.md`
    for suppression-free convergence of
    `telegram_runtime/session_preemption.rs` and updated marker-backlog counts.
127. Use `140-xiuxian-daochang-marker-burndown-wave-3-doc-driven-convergence-2026-02-27.md`
    for doc-driven marker burndown wave 3 in `xiuxian-daochang/tests`, including
    explicit crate-level test docs replacement, `agent/reflection` structural
    split for real `too_many_lines` removal, and updated backlog counts
    (`77/76` -> `42/41`) with strict clippy evidence.
128. Use `141-xiuxian-daochang-test-marker-zero-convergence-2026-02-27.md` for final
    `xiuxian-daochang/tests` convergence to zero
    `clippy::too_many_lines`/`clippy::too_many_arguments` marker occurrences,
    plus follow-up structural fixes from newly exposed real warnings.
129. Use `142-xiuxian-daochang-missing-docs-contract-and-settings-wave-2026-02-27.md`
    for production-source `missing_docs` reduction across
    `config/settings`, `contracts`, and agent state/metrics surfaces, plus
    environment-blocker evidence for Metal toolchain-dependent clippy runs.
130. Use `143-xiuxian-daochang-missing-docs-channel-policy-and-session-context-wave-2026-02-27.md`
    for `missing_docs` reduction in channel policy/ACL/session-context APIs
    and updated blocker-aware verification notes.
131. Use `144-xiuxian-daochang-missing-docs-runtime-config-and-session-gate-wave-2026-02-27.md`
    for incremental `missing_docs` reduction in runtime config loader,
    session-gate, webhook run/build requests, and markdown conversion APIs.
132. Use `145-xiuxian-daochang-xiuxian-config-docs-and-doc-markdown-followup-wave-2026-02-27.md`
    for `xiuxian-daochang` follow-up convergence on `config/xiuxian` public docs,
    suppression-free dead-code cleanup in unified config compatibility fields,
    and additional `doc_markdown` API/documentation cleanup.
133. Use `146-xiuxian-daochang-memory-recall-docs-and-test-function-split-followup-wave-2026-02-27.md`
    for additional suppression-free convergence: memory-recall public API docs,
    structural split of promoted-queue test flow to remove long-function lint
    pressure, and Omega `ReAct` doc-markdown follow-up cleanup.
134. Use `147-xiuxian-daochang-parser-and-gateway-warning-reduction-wave-2026-02-27.md`
    for warning-reduction convergence in Discord slash parser helper signatures,
    command/partition test function decomposition, and `examples/gateway` large-future
    mitigation via boxed awaits.
135. Use `148-xiuxian-daochang-warning-zero-convergence-across-pedantic-and-structural-lanes-2026-02-27.md`
    for end-to-end warning-zero convergence in `xiuxian-daochang` across pedantic and
    structural lanes (`too_many_lines`, `too_many_arguments`, `large_futures`),
    including dead-code cluster elimination in managed-runtime and telegram-media
    support integration targets.
136. Use `149-cross-crate-baseline-and-wendao-test-pedantic-convergence-2026-02-27.md`
    for second-pass cross-crate strict-clippy baseline validation
    (`xiuxian-qianji`, `xiuxian-zhixing`) and suppression-free pedantic warning
    cleanup in `xiuxian-wendao` resource-registry integration tests.
137. Use `150-xiuxian-qianhuan-expect-free-test-hardening-and-pedantic-cleanup-2026-02-27.md`
    for `xiuxian-qianhuan` convergence to the workspace no-`expect` standard in
    XML hardening tests, plus `manual_string_new` cleanup and targeted
    `cargo nextest` evidence (`12/12` passed).
138. Use `151-omni-events-unwrap-free-test-convergence-2026-02-27.md` for
    `omni-events` convergence to the workspace no-`unwrap` rule in broadcast
    async tests, including strict-clippy revalidation and crate-level
    `cargo nextest` evidence (`5/5` passed).
139. Use `152-omni-ast-strict-clippy-convergence-and-benchmark-stability-wave-2026-02-27.md`
    for suppression-free strict-clippy convergence in `omni-ast` test surfaces
    (`unwrap`/`expect` elimination + pedantic cleanup) and full crate
    `nextest` evidence (`89/89` passed).
140. Use `153-omni-tokenizer-cast-safety-and-benchmark-noise-hardening-wave-2026-02-27.md`
    for `omni-tokenizer` cast-safety convergence and benchmark-noise hardening
    (budgeted thresholds + configurable slack + lighter smoke iteration profile)
    with strict-clippy and full crate `nextest` evidence (`23/23` passed).
141. Use `154-xiuxian-zhenfa-and-omni-window-pedantic-convergence-wave-2026-02-27.md`
    for strict pedantic convergence in `xiuxian-zhenfa` and `omni-window`,
    including no-`expect_err` test migration, macro async/cache-key warning
    cleanup, doc-markdown alignment, and full crate `nextest` evidence
    (`28/28` + `3/3` passed).
142. Use `155-xiuxian-wendao-benchmark-budget-hardening-and-cross-crate-revalidation-wave-2026-02-27.md`
    for cross-crate strict-clippy/nextest revalidation
    (`xiuxian-qianji`, `xiuxian-zhixing`, `xiuxian-qianhuan`, `xiuxian-wendao`)
    and suppression-free hardening of wendao benchmark assertions via
    configurable CI/local budget slack (`OMNI_WENDAO_BENCH_SLACK_FACTOR`),
    with full `xiuxian-wendao` crate evidence (`286/286` passed, `1` skipped).
143. Use `156-xiuxian-wendao-cargo-benchmark-suppression-free-budget-hardening-wave-2026-02-28.md`
    for suppression-free convergence in `test_cargo_benchmark*`: removal of
    file-level `#![allow(...)]`, shared budget policy adoption
    (`OMNI_WENDAO_BENCH_SLACK_FACTOR` + `NEXTEST_RUN_ID` multiplier), and full
    strict-clippy/`nextest` evidence (`286` passed, `1` skipped).
144. Use `157-xiuxian-wendao-test-entrypoint-and-query-parsing-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction across `xiuxian-wendao` test entrypoints,
    core `mod.rs` roots, and `test_link_graph/query_parsing/*`, including
    follow-up `doc_markdown` root-cause fixes and strict-clippy/`nextest`
    evidence (`286` passed, `1` skipped) with burndown snapshot (`143 -> 116`).
145. Use `158-xiuxian-wendao-link-graph-filter-lanes-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in LinkGraph `search_filters` and
    `tree_scope_filters` test lanes, including `implicit_clone` root-cause
    fixes (`map_err(...to_string)` -> `map_err(...clone)`) and full
    strict-clippy/`nextest` evidence with burndown snapshot (`116 -> 99`).
146. Use `159-xiuxian-wendao-agentic-and-saliency-lanes-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in LinkGraph `agentic` and `saliency` test
    lanes, including root-cause fixes for `doc_markdown`, `implicit_clone`,
    `float_cmp`, `manual_string_new`, and `cast_lossless`, with full
    strict-clippy/`nextest` evidence and burndown snapshot (`99 -> 87`).
147. Use `160-xiuxian-wendao-link-graph-ppr-benchmark-lane-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in the LinkGraph PPR benchmark lane,
    including root-cause cleanup for `doc_markdown`, `format_push_string`, and
    cast-related percentile logic, with strict-clippy/`nextest` evidence and
    burndown snapshot (`87 -> 85`).
148. Use `161-xiuxian-wendao-wendao-cli-module-entry-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in `test_wendao_cli` module-entry files
    (`search/*` and `agentic/*` module roots), with strict-clippy/`nextest`
    evidence and burndown snapshot (`85 -> 80`).
149. Use `162-xiuxian-wendao-wendao-cli-related-and-directive-leaf-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in small `test_wendao_cli` related/directive
    leaf tests, with strict-clippy/`nextest` evidence and burndown snapshot
    (`80 -> 77`).
150. Use `163-xiuxian-wendao-wendao-cli-search-basic-and-directive-lanes-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction across `test_wendao_cli/search/basic` and
    `test_wendao_cli/search/directives` leaf tests, with strict-clippy/`nextest`
    evidence and burndown snapshot (`77 -> 69`).
151. Use `164-xiuxian-wendao-wendao-cli-search-link-filter-and-provisional-overlay-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in `test_wendao_cli/search` final leaf
    files (`link_filters` and `provisional_overlay`), with strict-clippy/`nextest`
    evidence and burndown snapshot (`69 -> 67`).
152. Use `165-xiuxian-wendao-knowledge-test-lane-allow-debt-reduction-and-doc-markdown-fix-wave-2026-02-28.md`
    for full `test_knowledge` lane suppression-debt reduction plus
    `doc_markdown` root-cause cleanup, with strict-clippy/`nextest` evidence
    and burndown snapshot (`67 -> 54`).
153. Use `166-xiuxian-wendao-sync-test-lane-allow-debt-reduction-and-pedantic-fixes-wave-2026-02-28.md`
    for full `test_sync` lane suppression-debt reduction, `uninlined_format_args`
    and `doc_markdown` root-cause fixes, and rerun-validated `nextest` evidence
    with burndown snapshot (`54 -> 46`).
154. Use `167-xiuxian-wendao-graph-traversal-and-entity-crud-allow-debt-reduction-wave-2026-02-28.md`
    for initial `test_graph` lane suppression-debt reduction (`entity_relation_crud`
    and `graph_traversal`) with `uninlined_format_args` root-cause fixes and
    strict-clippy/`nextest` evidence (`46 -> 44`).
155. Use `168-xiuxian-wendao-graph-lane-core-files-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in the remaining five core `test_graph`
    files, including `uninlined_format_args` and `float_cmp` root-cause
    cleanup, with strict-clippy/`nextest` evidence (`44 -> 39`).
156. Use `169-xiuxian-wendao-link-graph-cache-and-search-core-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in LinkGraph `cache_build` and `search_core`
    tests, including `implicit_clone` and float-assert root-cause cleanup, with
    strict-clippy/`nextest` evidence (`39 -> 37`).
157. Use `170-xiuxian-wendao-link-graph-build-scope-and-navigation-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in LinkGraph `build_scope` and
    `graph_navigation` tests, including `implicit_clone` and float-tolerance
    root-cause cleanup, with strict-clippy/`nextest` evidence (`37 -> 35`).
158. Use `171-xiuxian-wendao-link-graph-match-strategy-and-refresh-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in LinkGraph `search_match_strategies` and
    `refresh` tests, with `implicit_clone` root-cause cleanup and
    strict-clippy/`nextest` evidence (`35 -> 33`).
159. Use `172-xiuxian-wendao-link-graph-markdown-attachments-allow-debt-reduction-wave-2026-02-28.md`
    for final LinkGraph suppression-debt reduction in
    `markdown_attachments`, with `implicit_clone` root-cause cleanup and
    strict-clippy/`nextest` evidence (`33 -> 32`).
160. Use `173-xiuxian-wendao-hmas-and-kg-cache-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in top-level HMAS and KG-cache tests, with
    `doc_markdown` root-cause cleanup and strict-clippy/`nextest` evidence
    (`32 -> 30`).
161. Use `174-xiuxian-wendao-dependency-indexer-toml-only-config-migration-wave-2026-02-28.md`
    for dependency-indexer TOML-only config migration, including parser
    conversion, default-path migration from `references.yaml` to
    `xiuxian.toml`, test-fixture updates, and strict-clippy/`nextest`
    validation evidence.
162. Use `175-xiuxian-wendao-dependency-index-tests-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in dependency-index tests
    (`test_dependency_indexer` and `test_dependency_integration`), including
    `doc_markdown`, `uninlined_format_args`, and
    `needless_raw_string_hashes` root-cause cleanup with strict-clippy and
    `nextest` evidence (`28 -> 26`).
163. Use `176-xiuxian-wendao-intent-and-link-graph-refs-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in `test_intent` and
    `test_link_graph_refs`, including `doc_markdown` root-cause cleanup and
    strict-clippy/`nextest` evidence (`26 -> 24`).
164. Use `177-xiuxian-wendao-markdown-syntax-fixture-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in markdown syntax fixture tests, including
    `implicit_clone` root-cause cleanup and strict-clippy/`nextest` evidence
    (`24 -> 23`).
165. Use `178-xiuxian-wendao-link-graph-agentic-expansion-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in LinkGraph agentic-expansion integration
    tests, including clone-semantics root-cause cleanup and
    strict-clippy/`nextest` evidence (`23 -> 22`).
166. Use `179-xiuxian-wendao-ppr-precision-and-mixed-topology-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in PPR precision and mixed-topology graph
    tests, including `doc_markdown`, format-args, raw-string, and allocation
    root-cause cleanup with strict-clippy/`nextest` evidence (`22 -> 20`).
167. Use `180-xiuxian-wendao-seed-priors-ppr-and-performance-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in seed-priors module entrypoint, weighted
    PPR, and performance-stress tests, including `cast_lossless` root-cause
    cleanup with strict-clippy/`nextest` evidence (`20 -> 17`).
168. Use `181-xiuxian-wendao-agentic-overlay-module-entry-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in agentic overlay module-entry tests with
    strict-clippy/`nextest` evidence (`17 -> 16`).
169. Use `182-xiuxian-wendao-agentic-verbose-and-seed-priors-journal-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in agentic verbose execution and
    seed-priors journal-linked scenario tests, including formatting, clone, and
    raw-string root-cause cleanup with strict-clippy/`nextest` evidence
    (`16 -> 14`).
170. Use `183-xiuxian-wendao-seed-priors-architecture-hub-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in the structural-priors architecture-hub
    scenario test, including clone and format-args root-cause cleanup with
    strict-clippy/`nextest` evidence (`14 -> 13`).
171. Use `184-xiuxian-wendao-seed-priors-related-filter-accuracy-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in seed-grounded related-filter accuracy
    scenario tests, including fixture-loop compaction and clone/format root-cause
    cleanup with strict-clippy/`nextest` evidence (`13 -> 12`).
172. Use `185-xiuxian-wendao-agentic-persist-suggestions-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in the agentic persist-suggestions CLI
    scenario test, including helper extraction and format/line-count structural
    cleanup with strict-clippy/`nextest` evidence (`12 -> 11`).
173. Use `186-xiuxian-wendao-overlay-alias-resolution-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in the mixed-alias promoted-overlay
    scenario, including shared overlay helper extraction and line-count
    structural cleanup with strict-clippy/`nextest` evidence (`11 -> 10`).
174. Use `187-xiuxian-wendao-attachments-cli-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in attachments CLI tests, including command
    helper extraction and line-count structural cleanup with
    strict-clippy/`nextest` evidence (`10 -> 9`).
175. Use `188-xiuxian-wendao-agentic-discovery-quality-signals-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in the agentic discovery-quality scenario,
    including shared execution helper extraction and line-count structural
    cleanup with strict-clippy/`nextest` evidence (`9 -> 8`).
176. Use `189-xiuxian-wendao-overlay-key-prefix-isolation-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in the key-prefix isolation overlay
    scenario, including helper reuse and line-count structural cleanup with
    strict-clippy/`nextest` evidence (`8 -> 7`).
177. Use `190-xiuxian-wendao-agentic-log-flow-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction in the agentic log/recent/decide flow
    scenario, including shared helper extraction in `agentic/mod.rs` and
    strict-clippy/`nextest` evidence (`7 -> 6`).
178. Use `191-xiuxian-wendao-related-module-split-and-allow-debt-reduction-wave-2026-02-28.md`
    for suppression-debt reduction and interface-only modularization in the
    `related` test module, including extracted diagnostics/monitor assertion
    files and strict-clippy/`nextest` evidence (`6 -> 5`).
179. Use `192-xiuxian-wendao-cli-final-allow-debt-zero-convergence-wave-2026-02-28.md`
    for final suppression-debt elimination in `xiuxian-wendao/tests`,
    including helper-driven decomposition of long CLI scenario tests and full
    strict-clippy/`nextest` evidence (`5 -> 0`).
180. Use `193-xiuxian-qianhuan-contracts-missing-docs-suppression-removal-wave-2026-02-28.md`
    for suppression-free convergence of the `xiuxian-qianhuan` contracts test
    entrypoint (`tests/contracts.rs`), replacing file-level `missing_docs`
    allowance with explicit module documentation and full strict-clippy/`nextest`
    evidence (`1 -> 0`).
181. Use `194-omni-memory-test-missing-docs-suppression-removal-wave-2026-02-28.md`
    for suppression-free convergence of remaining `omni-memory` test entry
    files, replacing file-level `missing_docs` allowances with explicit module
    documentation and full strict-clippy/`nextest` evidence (`2 -> 0`).
182. Use `195-xiuxian-qianji-missing-docs-only-test-suppression-reduction-wave-2026-02-28.md`
    for targeted reduction of `xiuxian-qianji` missing-docs-only test
    suppressions, replacing file-level allowances with module docs and
    revalidating with strict-clippy/`nextest` evidence (`17 -> 12`).
183. Use `196-xiuxian-qianji-doc-markdown-test-suppression-reduction-wave-2026-02-28.md`
    for suppression reduction in `xiuxian-qianji` test files that carried
    `missing_docs + doc_markdown` allowances, with documentation-first cleanup
    and strict-clippy/`nextest` evidence (`12 -> 8`).
184. Use `197-xiuxian-qianji-test-allow-debt-zero-convergence-wave-2026-02-28.md`
    for final `xiuxian-qianji` test suppression convergence, including
    removal of remaining mixed `allow` attributes, root-cause unused-import
    cleanup, and strict-clippy/`nextest` evidence (`8 -> 0`).
185. Use `198-xiuxian-llm-test-allow-debt-zero-convergence-wave-2026-02-28.md`
    for final `xiuxian-llm` test suppression convergence, including removal
    of all file-level `allow` attributes, follow-up root-cause clippy fixes
    (`manual_let_else`, float tolerance assertions, async-fn conversion, and
    no-effect binding cleanup), and strict-clippy/`nextest` evidence (`16 -> 0`).
186. Use `199-xiuxian-daochang-tests-allow-debt-zero-and-settings-merge-compat-wave-2026-02-28.md`
    for `xiuxian-daochang` test-lane file-level allow-debt elimination (`52 -> 0`)
    plus settings-merge schema compatibility repair for
    `agenda_validation_policy`, with strict-clippy evidence and explicit
    `nextest` failure inventory for follow-up runtime-regression closure.
187. Use `200-xiuxian-daochang-nextest-regression-closure-agenda-validation-isolation-wave-2026-02-28.md`
    for post-wave regression closure in `xiuxian-daochang` tests (`9 -> 0`), including
    test-fixture policy isolation (`agenda_validation_policy = "never"`),
    notebook state isolation for `native_tools_zhixing_e2e`, embedding retry
    assertion alignment, and full-crate `nextest` + strict-clippy evidence.
188. Use `201-xiuxian-daochang-large-futures-convergence-and-clippy-unblock-wave-2026-02-28.md`
    for `xiuxian-daochang` large-future convergence via source-side boxing in
    `run_turn`, plus workspace-clippy unblock fixes in `xiuxian-skills` and
    `xiuxian-zhixing`, with strict-clippy and full `nextest` evidence
    (`653 passed`, `0 failed`).
189. Use `202-xiuxian-daochang-src-allow-cleanup-root-cause-reflection-and-observability-wave-2026-02-28.md`
    for suppression-free convergence in `xiuxian-daochang/src` dead-code and
    unused-import lanes, including reflection/runtime call-path fixes,
    Telegram test-scope export cleanup, legacy metadata decode observability,
    and strict-clippy + full `nextest` evidence (`653 passed`, `0 failed`).
190. Use `203-xiuxian-daochang-env-free-bootstrap-tests-and-zhenfa-reload-args-compat-wave-2026-02-28.md`
    for `xiuxian-daochang` bootstrap/config test hardening without process-global env
    mutation (`unsafe_code` allowance removal path), plus native zhenfa
    reload-argument compatibility convergence across `xiuxian-daochang` and
    `xiuxian-qianhuan`, with strict-clippy and full `nextest` evidence
    (`653 passed`, `0 failed`).
191. Use `204-xiuxian-daochang-src-final-allow-zero-embedding-dimension-cast-safety-wave-2026-02-28.md`
    for final `xiuxian-daochang/src` allow-zero convergence by removing the last
    cast-related suppression in `embedding_dimension.rs` through explicit
    numeric conversion and bounds-safe interpolation, validated by strict-clippy
    and full `nextest` evidence (`653 passed`, `0 failed`).
192. Use `205-package-rename-follow-up-audit-and-dependency-chain-verification-wave-2026-02-28.md`
    for package-rename follow-up verification, including old-name residue
    audit (`omni-config-core`, `omni-macros`, `omni-scanner`), workspace and
    metadata confirmation for renamed crates (`xiuxian-*`), and dependency-chain
    compile evidence across renamed crates and key dependents.
193. Use `206-rename-follow-up-test-lint-convergence-for-macros-and-wendao-wave-2026-02-28.md`
    for immediate post-rename clippy/nextest convergence in
    `xiuxian-macros` and `xiuxian-wendao`, including root-cause replacement of
    test `unwrap/expect` usage and validated clean outcomes without lint
    suppression.
194. Use `207-xiuxian-skills-test-lint-debt-burndown-wave-2026-02-28.md`
    for ongoing strict-clippy debt burndown in `xiuxian-skills`, including
    broad `unwrap/expect` elimination across multiple test modules and
    convergence of remaining failures to the `tools_scanner` lane
    (`205 -> 154` errors in that target slice).
195. Use `208-xiuxian-skills-tools-scanner-strict-clippy-convergence-wave-2026-02-28.md`
    for final strict-clippy convergence in the historical `tools_scanner`
    lane of `xiuxian-skills`, including full `unwrap` removal in the remaining
    tests, warning cleanup (`doc_markdown` and needless borrows), and validated
    `nextest` + strict-clippy evidence (`189/189` crate tests and `45/45`
    `tools_scanner` tests passed).
196. Use `209-xiuxian-skills-skill-scanner-warning-zero-convergence-wave-2026-02-28.md`
    for warning-zero convergence in `xiuxian-skills/tests/skill_scanner.rs`,
    including doc-markdown cleanup, removal of needless borrows and
    `items_after_statements` patterns, and verified `nextest` + strict-clippy
    evidence (`15/15` lane tests and `189/189` crate tests passed).
197. Use `210-xiuxian-skills-schema-validation-warning-zero-convergence-wave-2026-02-28.md`
    for warning-zero convergence in `xiuxian-skills/tests/test_schema_validation.rs`,
    including schema-doc markdown normalization, borrow/format/assert cleanup,
    and strict validation evidence (`12/12` lane tests passed; crate warning
    baseline `132 -> 94`).
198. Use `211-xiuxian-skills-test-skill-scanner-warning-zero-convergence-wave-2026-02-28.md`
    for warning-zero convergence in `xiuxian-skills/tests/test_skill_scanner.rs`,
    including doc markdown cleanup, fixture default/style normalization, and
    full-lane plus full-crate verification evidence (`17/17` lane tests,
    `189/189` crate tests; crate warning baseline `94 -> 60`).
199. Use `212-xiuxian-skills-warning-zero-convergence-final-wave-2026-02-28.md`
    for the final strict-clippy warning-zero convergence in `xiuxian-skills`,
    covering the remaining lanes (`test_full_workflow`, `test_skill_metadata`,
    `test_skill_structure_config_cascade`, `test_schema_generation`,
    `test_benchmark`, and lib tests) with full verification evidence
    (`0 warnings`, `189/189` tests passed).
200. Use `213-zhixing-resource-image-export-and-indexer-test-modularization-wave-2026-03-01.md`
    for the Zhixing resource-image export and bootstrap de-legacy wave,
    including removal of `embed_utf8_dir!` from `xiuxian-daochang` Zhixing bootstrap,
    canonical `RESOURCES` export in `xiuxian-zhixing`, Wendao in-memory `SKILL.md`
    extraction test evidence, and closure of the remaining
    `clippy::too_many_lines` warning in `xiuxian-zhixing` test lane.
201. Use `214-skill-vfs-native-memory-bus-phase-17-2-followup-2026-03-01.md`
    for the Phase 17.2 Skill VFS follow-up: explicit embedded mounting,
    memory-backed semantic reads via `Arc<str>`, canonical URI cache keys,
    one-time embedded mount indexing, and cross-crate validation evidence
    (`xiuxian-wendao`, `xiuxian-qianji`, `xiuxian-qianhuan`, and `xiuxian-daochang`).
202. Use `215-skill-vfs-mounted-resource-registry-and-interning-2026-03-01.md`
    for the ADR-007 mount-registry convergence wave: resolver-owned embedded
    resource mount map (`HashMap<String, Dir>` model), concurrent string
    interning via `DashMap<String, Arc<str>>`, direct `Dir::get_file` semantic
    reads, and downstream compatibility validation across Qianhuan/Qianji lanes.
203. Use `216-xiuxian-daochang-src-tests-relocation-to-top-level-tests-unit-2026-03-01.md`
    for the structural test-layout normalization in `xiuxian-daochang`: relocating
    `src/**/tests.rs` implementations to `tests/unit/**`, updating module path
    mounts, and preserving embedded SkillVFS startup/bridge contracts with
    strict validation evidence.
204. Use `217-cross-crate-src-inline-tests-relocation-and-lint-convergence-2026-03-01.md`
    for cross-crate structural normalization of test layouts (removing inline
    `src` test implementations and relocating to `tests/unit/**`), strict
    clippy convergence without suppression-based shortcuts, and targeted
    nextest evidence over touched lanes.
205. Use `218-omni-tui-warning-reduction-wave-2026-03-01.md`
    for the post-convergence warning-reduction slice in `omni-tui`, including
    targeted `must_use` adoption, doc-markdown cleanup, format-string
    normalization, and warning baseline reduction with strict clippy + targeted
    nextest evidence.
206. Use `219-omni-tui-warning-zero-and-test-layout-guard-2026-03-01.md`
    for final warning-zero convergence in the touched `omni-tui` slice and
    introduction of a CI-wired Rust test-layout guard that prevents inline
    `src` test implementations and broken `#[path]` mounts from regressing.
207. Use `220-xiuxian-qianji-test-consensus-and-agenda-lint-convergence-2026-03-01.md`
    for strict-clippy convergence in `xiuxian-qianji` test lanes (`unwrap`
    removal and agenda-test modularization), plus cross-crate strict clippy
    revalidation evidence.
208. Use `221-cache-test-api-stale-dead-code-allow-removal-2026-03-01.md`
    for stale suppression cleanup in `xiuxian-vector` and `xiuxian-wendao` cache
    test-helper APIs (`#[allow(dead_code)]` removal), with strict clippy,
    targeted `nextest`, and test-layout guard revalidation evidence.
209. Use `222-xiuxian-qianji-cast-and-llm-test-hardening-2026-03-01.md`
    for `xiuxian-qianji` production cast-suppression removal, Python-module
    runtime unwrap elimination, LLM test-lane `expect` removal, and all-features
    clippy recovery with targeted nextest validation.
210. Use `223-xiuxian-qianji-python-module-and-llm-tests-contract-alignment-2026-03-01.md`
    for all-features warning cleanup in `xiuxian-qianji` Python bindings,
    deterministic LLM test-lane hardening, and agenda-workflow assertion
    alignment to current embedded manifest output contract.
211. Use `224-xiuxian-qianji-error-enum-de-suffix-and-final-enum-variant-allow-removal-2026-03-01.md`
    for removing the final `enum_variant_names` suppression in
    `xiuxian-qianji/src` through `QianjiError` variant de-suffixing and full
    strict-clippy plus LLM integration revalidation.
212. Use `225-omni-io-assembler-allow-removal-and-feature-lane-hardening-2026-03-01.md`
    for `omni-io` assembler suppression removal via borrowed/generic API and
    explicit typed main-file error path, plus strict clippy and `assembler`
    feature-lane `nextest` evidence.
213. Use `226-omni-sandbox-test-lane-warning-zero-and-executor-name-lint-cleanup-2026-03-01.md`
    for `omni-sandbox` executor-name suppression removal (`unused_self`) and
    strict-clippy warning-zero convergence in `tests/test_sandbox.rs` through
    top-level integration-test normalization and `expect`-free tempdir setup,
    with targeted `nextest` evidence (`17 passed`, `0 failed`).
214. Use `227-omni-sandbox-src-allow-zero-convergence-and-bool-flag-modeling-2026-03-01.md`
    for final source-level suppression removal in `omni-sandbox` (`unsafe_derive_deserialize`
    and `struct_excessive_bools`) via pyclass deserialize-boundary cleanup and
    typed `BoolFlag` modeling in nsjail config, with strict-clippy and targeted
    nextest revalidation evidence (`17 passed`, `0 failed`).
215. Use `228-omni-memory-pybindings-allow-removal-and-src-allow-zero-milestone-2026-03-01.md`
    for `omni-memory` pybindings suppression-block removal through structural
    and documentation fixes, `float_cmp` cleanup in feedback-tracking tests,
    strict clippy and full nextest evidence (`69 passed`, `0 failed`), and
    repository milestone confirmation that `packages/rust/crates/*/src` has
    zero `#[allow(...)]` matches.
216. Use `229-test-structure-wrapper-zero-and-xiuxian-mcp-dead-code-allow-removal-2026-03-01.md`
    for test-layout normalization in `omni-ast` (`tests` file wrapper removal)
    and dead-code suppression removal in `xiuxian-mcp` streamable-http
    integration test via active real-port probing, with strict clippy and
    targeted nextest evidence (`xiuxian-mcp`: `1 passed, 1 skipped`;
    `omni-ast --lib`: `48 passed`).
217. Use `230-xiuxian-daochang-telegram-media-test-support-no-allow-modular-mount-2026-03-01.md`
    for `xiuxian-daochang` Telegram media integration-test support refactoring:
    per-target `#[path]` module mounts, elimination of suppression-driven
    wrapper patterns, explicit helper-submodule path stabilization, and
    strict-clippy + targeted nextest evidence (`25 passed`, `0 skipped`) across
    all touched Telegram media test binaries.
218. Use `231-xiuxian-daochang-all-target-clippy-unblock-via-valkey-hooks-and-wendao-refresh-import-fix-2026-03-01.md`
    for unblocking `xiuxian-daochang` all-target strict clippy by replacing
    `expect_err` in valkey hook tests with explicit error matching and aligning
    `xiuxian-qianji` LinkGraph refresh imports to current `xiuxian-wendao`
    public paths, with successful all-target clippy completion evidence.
219. Use `232-xiuxian-daochang-all-target-warning-reduction-followup-and-targeted-nextest-proof-2026-03-01.md`
    for follow-up warning reduction after the all-target unblock, including
    test-function decomposition in valkey live hooks, removal of unnecessary
    `Result` wrapping in embedded-defaults probe tests, and targeted nextest
    evidence for both touched lanes while keeping all-target strict clippy
    green.
220. Use `233-omni-mcp-client-test-allow-zero-with-real-port-probe-and-workspace-gate-note-2026-03-01.md`
    for final cleanup of test-tree `#[allow(dead_code)]` debt in
    `omni-mcp-client` streamable-http integration tests via active real-port
    probing, plus explicit documentation of the current workspace-metadata
    blocker preventing crate-local `clippy`/`nextest` execution.
221. Use `234-xiuxian-daochang-runtime-agent-factory-tests-top-level-harness-migration-2026-03-01.md`
    for migrating `runtime_agent_factory` tests out of `src`-side
    `#[cfg(test)]` mounting into a package-top integration harness, including
    strict-clippy and nextest proof (`18 passed`, `0 failed`) for the migrated
    lane.
222. Use `235-xiuxian-daochang-llm-and-embedding-tests-top-level-harness-convergence-2026-03-01.md`
    for converging `xiuxian-daochang` `llm` and `embedding` lanes to package-top
    `tests/` harnesses, removing remaining `src`-side test mounts, resolving
    harness compile/lint blockers without suppression, and validating with
    strict clippy plus targeted nextest (`llm`: `14 passed`; `embedding`:
    `8 passed`).
223. Use `236-xiuxian-daochang-nodes-warmup-and-config-xiuxian-tests-top-level-harness-migration-2026-03-01.md`
    for migrating `nodes/warmup` and `config/xiuxian` lanes from `src`-side
    path mounts to package-top harnesses, including minimal harness-side
    dependency shims, warning-zero strict clippy, and targeted nextest proof
    (`nodes_warmup`: `3 passed`; `config_xiuxian`: `2 passed`).
224. Use `237-xiuxian-daochang-mcp-startup-tests-top-level-harness-migration-2026-03-01.md`
    for migrating `agent/mcp_startup` from `src`-side test mount to a
    package-top harness with explicit dependency surface, warning-zero strict
    clippy, and targeted nextest proof (`2 passed`), plus aggregated migrated-lane
    revalidation (`15 passed` across `embedding`, `nodes_warmup`,
    `config_xiuxian`, `agent_mcp_startup`).
225. Use `238-xiuxian-daochang-memory-recall-and-admission-tests-top-level-harness-migration-2026-03-01.md`
    for migrating `memory_recall_feedback`, `memory_recall_metrics`, and
    `admission` out of `src`-side path mounts into package-top harnesses with
    structural dependency stubs, warning-zero strict clippy, and aggregated
    nextest proof (`22 passed`, `0 failed` across the three migrated lanes),
    leaving only `agent/bootstrap.rs` as the final remaining `src` path-mount.
226. Use `239-xiuxian-daochang-bootstrap-tests-top-level-harness-migration-and-src-path-mount-zero-2026-03-01.md`
    for removing the final `src`-side bootstrap path mount, introducing the
    package-top `agent_bootstrap` harness, and proving `xiuxian-daochang/src`
    path-mount zero (`rg` no matches) with strict clippy and aggregated nextest
    evidence (`40 passed`, `0 failed` across migrated lanes).
227. Use `240-cross-crate-src-path-mount-zero-convergence-omni-tui-xiuxian-vector-qianji-wendao-2026-03-02.md`
    for completing cross-crate `src` path-mount elimination in `omni-tui`,
    `xiuxian-vector`, `xiuxian-qianji`, and `xiuxian-wendao`, with package-top
    harness migration, zero-suppression structural fixes, global `rg`
    no-match proof, and strict clippy + targeted nextest evidence across all
    touched crates (`42 + 9 + 3 + 36 passed`, `0 failed`).
228. Use `241-src-test-path-mount-burndown-wave-2-infra-crates-and-indexer-lanes-2026-03-02.md`
    for the next `src` path-mount reduction wave across infra/support crates
    (`omni-events`, `omni-executor`, `omni-io`, `omni-security`, `omni-tui`,
    `xiuxian-vector`, `xiuxian-memory`, and `xiuxian-wendao` dependency indexer
    lanes), including feature-matrix nextest validation and updated global
    remaining-count evidence (`41` path mounts left in `src`).
229. Use `242-xiuxian-skills-src-path-mount-zero-convergence-and-harness-shim-stabilization-2026-03-02.md`
    for the `xiuxian-skills` `src` path-mount zero convergence wave, including
    removal of 14 `src`-side path mounts, package-top harness migration with
    shim stabilization for legacy `super`/`crate` imports, and strict clippy
    plus targeted nextest evidence (`80 passed`, `0 failed`) with global
    remaining-count update (`18` path mounts left in `src`).
230. Use `243-omni-tui-and-xiuxian-vector-src-path-mount-elimination-wave-2026-03-02.md`
    for eliminating the remaining `src` path mounts in `omni-tui` and
    `xiuxian-vector`, including package-top harness migration, `omni-tui` CLI
    args module extraction for warning-free argument tests, and strict clippy
    plus targeted nextest evidence (`3 + 10` passed, `0` failed) with global
    remaining-count update (`15` path mounts left in `src`).
231. Use `244-xiuxian-daochang-memory-and-reflection-src-path-mount-elimination-wave-2026-03-02.md`
    for the next `xiuxian-daochang` migration wave removing `src`-side mounts from
    `memory::decay`, `memory::recall_credit`, `memory_recall`, and
    `reflection`, including package-top harnesses, targeted nextest proof
    (`17 passed`, `0 failed`), and updated global remaining-count evidence
    (`11` path mounts left in `src`), with explicit clippy blocker recording
    for unrelated `xiuxian-qianji` compile errors.
232. Use `245-xiuxian-daochang-gateway-runtime-src-path-mount-elimination-2026-03-02.md`
    for eliminating the `src`-side mount in `gateway/http/runtime`, adding the
    package-top `gateway_http_runtime_unit` harness, and validating both lane
    and aggregate migrated suites (`9` and `26` tests passed, `0` failed) with
    updated global remaining-count evidence (`10` path mounts left in `src`).
233. Use `246-xiuxian-daochang-llm-proxy-and-redis-message-store-src-path-mount-elimination-2026-03-02.md`
    for eliminating `src`-side mounts in `gateway/http/llm_proxy` and
    `session/redis_backend`, adding package-top harnesses
    (`gateway_http_llm_proxy_unit`, `session_redis_backend_unit`), and
    validating with targeted nextest (`11 passed`, `0 failed`) plus strict
    clippy evidence, with updated global remaining-count evidence (`8` path
    mounts left in `src`).
234. Use `247-xiuxian-daochang-managed-runtime-and-memory-stream-consumer-src-path-mount-elimination-2026-03-02.md`
    for eliminating `src`-side mounts in `channels::managed_runtime` and
    `agent::memory_stream_consumer`, adding package-top harnesses
    (`channels_managed_runtime_unit`, `agent_memory_stream_consumer_unit`),
    and validating with targeted nextest (`21 passed`, `0 failed`, `4 skipped`)
    plus strict clippy evidence, with updated global remaining-count evidence
    (`6` path mounts left in `src`).
235. Use `248-xiuxian-daochang-final-src-path-mount-zero-convergence-telegram-runtime-harness-wave-2026-03-02.md`
    for the final convergence wave eliminating the remaining
    `channels::telegram::runtime` `src` path mount, introducing the package-top
    `channels_telegram_runtime_unit` harness, and proving global `src`
    path-mount zero with `rg` no-match evidence, aggregated migrated-lane
    nextest proof (`104 passed`, `0 failed`), and mandatory strict clippy
    validation for `xiuxian-daochang`.
236. Use `249-xiuxian-daochang-src-cfg-test-zero-convergence-and-config-test-migration-2026-03-02.md`
    for eliminating all remaining `#[cfg(test)]` usage under `xiuxian-daochang/src`,
    migrating `config/tests_xiuxian.rs` from `src` into package-top
    `tests/unit/config`, updating harness paths without suppression-based
    workarounds, and validating with targeted nextest (`108 passed`, `0 failed`,
    `4 skipped`; plus follow-up touched-lane run `65 passed`, `0 failed`,
    `2 skipped`) plus mandatory clippy gate success.
237. Use `250-xiuxian-daochang-discord-runtime-harness-warning-reduction-wave-2026-03-02.md`
    for the warning-reduction convergence wave in
    `channels_discord_runtime_unit` and `agent_memory_recall_state_unit`,
    including structural harness symbol probes (no `allow` suppression),
    dead-code shim cleanup, warning-clean targeted lane validation
    (`32 passed`, `0 failed`, `2 skipped`), wider touched-lane regression
    (`65 passed`, `0 failed`, `2 skipped`), and mandatory clippy gate success.
238. Use `251-xiuxian-daochang-telegram-runtime-harness-warning-zero-convergence-wave-2026-03-02.md`
    for warning-zero convergence in `channels_telegram_runtime_unit`,
    including harness-local structural symbol probes across managed-command,
    managed-runtime, and telegram runtime surfaces (no `allow` suppression),
    reduction from `60` compile warnings to zero, targeted nextest proof
    (`74 passed`, `0 failed`, `0 skipped`), and mandatory clippy gate success.
239. Use `252-xiuxian-skills-top-level-tests-path-attr-removal-and-module-normalization-2026-03-02.md`
    for removing package-top test `#[path = "..."]` remapping in
    `xiuxian-skills` `skills_tools_unit`, normalizing the module tree directly
    under `tests/skills_tools_tests`, and validating with targeted nextest
    (`21 passed`, `0 failed`) plus mandatory clippy gate success.
240. Use `253-xiuxian-skills-test-include-indirection-reduction-wave-2026-03-02.md`
    for reducing `include!("unit/...")` indirection across 11
    `xiuxian-skills` package-top harnesses by switching to standard `mod tests;`
    modules with moved test sources under `tests/*_module/tests.rs`, including
    targeted nextest validation (`51 passed`, `0 failed`) and mandatory clippy
    gate success, with 2 complex include-based lanes explicitly deferred.
241. Use `254-xiuxian-skills-virtual-path-filter-harness-retirement-and-include-reduction-2026-03-02.md`
    for retiring the redundant include-heavy
    `skills_tools_scan_virtual_paths_unit` lane (coverage preserved in
    `skills_tools_unit` scan-path tests), deleting its legacy unit file,
    validating with adjacent-lane regression (`77 passed`, `0 failed`) plus
    mandatory clippy gate success, and reducing remaining include-based
    harnesses in `xiuxian-skills/tests` to one deferred lane.
242. Use `255-xiuxian-skills-references-harness-public-api-rewrite-and-include-zero-2026-03-02.md`
    for rewriting `skills_scanner_references_unit` from `../src/...` include
    mounts to public-API integration tests (`build_index_entry`/`scan_skill`),
    deleting the final legacy `tests/unit` references file, validating with
    adjacent-lane regression (`77 passed`, `0 failed`) plus mandatory clippy
    gate success, and achieving include-zero across
    `packages/rust/crates/xiuxian-skills/tests`.
243. Use `256-xiuxian-vector-and-xiuxian-tui-test-include-indirection-reduction-wave-2026-03-02.md`
    for reducing `unit/...` include indirection in 3 harnesses across
    `xiuxian-tui` and `xiuxian-vector`, moving test sources into module-local
    `tests/*.rs`, validating with targeted nextest (`3` and `10` tests passed)
    plus mandatory clippy gates for both touched crates, achieving include-zero
    in `xiuxian-tui/tests`, and reducing `xiuxian-vector/tests` include usage to
    2 explicit source seam mounts.
244. Use `257-xiuxian-daochang-config-xiuxian-test-include-indirection-reduction-2026-03-02.md`
    for normalizing `xiuxian-daochang` `config_xiuxian` harness from
    `unit/config/*.rs` include mounts to module-local `tests/config/*.rs`,
    validating with targeted nextest (`12 passed`) and touched-target clippy
    gate success (`--test config_xiuxian`), while documenting remaining
    pre-existing upstream warnings outside this harness migration scope.
245. Use `258-xiuxian-daochang-unit-include-indirection-zero-convergence-wave-2026-03-02.md`
    for the final `tests/unit/**` include-indirection convergence in
    `xiuxian-daochang/tests`, completing migration of managed-runtime, bootstrap,
    zhenfa, session-context, and redis-backend harnesses to module-local
    package-top directories, with structural `rg` no-match proof plus targeted
    nextest validation (`7` + `39` passed, `0` failed, `1` skipped) and
    mandatory touched-crate clippy gate success.
246. Use `259-xiuxian-daochang-tests-local-include-indirection-zero-convergence-2026-03-02.md`
    for full local include-indirection convergence in `xiuxian-daochang/tests`
    beyond `unit/**`, including agent/domain, gateway/nodes, and
    discord/telegram runtime harness migrations to module-local paths with
    structural no-match proof (`include!("...")` filtered by non-`../src`),
    targeted nextest evidence (`57` + `19` + `102` passed, `0` failed), and
    mandatory touched-crate clippy gate success.
247. Use `260-xiuxian-daochang-and-xiuxian-llm-clippy-warning-reduction-wave-2026-03-02.md`
    for a warning-reduction wave focused on `xiuxian-daochang` and `xiuxian-llm`,
    including provider-mode simplification, litellm runtime field-name cleanup,
    provider boundary tightening, inference dead-code path activation, provider
    doc-markdown cleanup, and feature-disabled async fallback convergence, with
    targeted nextest evidence (`20` + `17` + `68` passed, `0` failed) and
    mandatory touched-crate clippy gate success for both crates.
248. Use `261-xiuxian-macros-gate-closure-and-xiuxian-tui-inline-test-elimination-2026-03-02.md`
    for closing the `xiuxian-macros` `too_many_lines` convergence gate,
    proving transitive cleanliness via `xiuxian-daochang --tests` strict clippy lane,
    and removing the last inline test from `xiuxian-tui` `src/examples` by
    migrating demo CLI parsing coverage to package-top `tests`, with targeted
    nextest proof (`4 passed`, `0 failed`) and mandatory touched-crate clippy
    gate success.
249. Use `262-xiuxian-wendao-fusion-public-api-test-decoupling-and-warning-cleanup-2026-03-02.md`
    for removing `fusion_unit` source-path remapping in `xiuxian-wendao`,
    exporting fusion computation API at crate boundary, cleaning newly surfaced
    pedantic/doc warnings in `src/fusion.rs`, deleting obsolete dead
    `tests/unit/fusion_tests.rs`, and validating with targeted nextest
    (`2 passed`, `0 failed`) plus mandatory touched-crate clippy gate success
    (warning-free).
250. Use `263-xiuxian-daochang-observability-test-decoupling-and-session-event-api-stabilization-2026-03-02.md`
    for replacing `xiuxian-daochang` observability test source remapping with a
    stable public `session_event_ids()` accessor, keeping `SessionEvent`
    internal to avoid public-doc debt, and validating with targeted nextest
    (`4 passed`, `0 failed`) plus mandatory touched-crate clippy gate success
    on changed files (with unrelated dependency warnings explicitly scoped).
251. Use `264-xiuxian-daochang-config-xiuxian-test-remap-elimination-and-public-loader-alignment-2026-03-02.md`
    for removing `config_xiuxian` path remap shim usage in `xiuxian-daochang/tests`,
    promoting xiuxian path/base loader helpers to public crate contract,
    deleting obsolete `tests/config/xiuxian.rs`, and validating with targeted
    nextest (`13 passed`, `0 failed`) plus mandatory touched-crate clippy gate
    success.
252. Use `265-xiuxian-vector-match-util-public-export-and-test-remap-elimination-2026-03-02.md`
    for removing `keyword_fusion_match_util_unit` source remapping in
    `xiuxian-vector/tests`, exposing fusion match-util helpers through the
    `keyword::fusion` public boundary with warning-clean annotations/docs, and
    validating with targeted nextest (`4 passed`, `0 failed`) plus mandatory
    touched-crate clippy gate success.
253. Use `266-xiuxian-wendao-link-graph-refresh-test-public-api-migration-2026-03-02.md`
    for removing `link_graph_refresh_unit` private strategy source remapping in
    `xiuxian-wendao/tests`, rewriting assertions to public
    `LinkGraphIndex::refresh_incremental_with_threshold` behavior (`noop/full/delta`),
    and validating with targeted nextest (`3 passed`, `0 failed`) plus
    mandatory touched-crate clippy gate success.
254. Use `267-xiuxian-daochang-runtime-agent-factory-test-remap-elimination-and-lib-boundary-alignment-2026-03-02.md`
    for removing `runtime_agent_factory` source remapping from
    `xiuxian-daochang/tests`, introducing stable `test_support` wrappers, aligning
    runtime-agent build ownership to the library boundary via
    `xiuxian_daochang::build_agent`, and validating with targeted nextest (`20` +
    `23` passed, `0` failed) plus mandatory touched-crate clippy gate success.
255. Use `268-xiuxian-daochang-memory-recall-and-reflection-test-remap-elimination-2026-03-02.md`
    for removing source remapping from `agent_memory_recall_unit` and
    `agent_reflection_unit`, adding stable memory-recall/reflection wrappers in
    `xiuxian_daochang::test_support`, and validating with targeted nextest
    (`11 passed`, `0 failed`) plus mandatory touched-crate clippy gate success.
256. Use `269-xiuxian-daochang-discord-runtime-test-remap-elimination-2026-03-02.md`
    for removing source remapping from `channels_discord_runtime_unit`,
    replacing trait/managed-command/session-partition dependencies with
    public/test-support boundaries, and validating with targeted nextest
    (`28 passed`, `0 failed`) plus mandatory touched-crate clippy gate success.
257. Use `270-xiuxian-daochang-telegram-runtime-test-remap-reduction-wave-2026-03-02.md`
    for reducing source remaps in `channels_telegram_runtime_unit` by
    migrating managed-command and telegram-command dependencies to
    `test_support`, validating with targeted nextest (`74 passed`, `0 failed`)
    and mandatory clippy gate success, while recording the single remaining
    `runtime_config` remap blocker caused by queue-mode type coupling.
258. Use `271-xiuxian-daochang-telegram-runtime-test-remap-zero-convergence-2026-03-02.md`
    for closing the final `runtime_config` remap in
    `channels_telegram_runtime_unit` via a local adapter that bridges queue-mode
    type boundaries, with targeted nextest proof (`74 passed`, `0 failed`) and
    mandatory clippy gate success.
259. Use `272-xiuxian-daochang-llm-test-remap-elimination-and-zero-convergence-2026-03-02.md`
    for removing the final `llm` source remaps by introducing a stable
    `llm/test_api + test_support` boundary, validating with targeted nextest
    (`23 passed`, `0 failed`) and mandatory clippy gate success, and proving
    zero remap matches across all `xiuxian-daochang/tests`.
260. Use `273-xiuxian-llm-vision-pedantic-warning-convergence-2026-03-02.md`
    for closing `xiuxian-llm` vision pedantic warnings via root-cause fixes
    (integer rounding resize math, `f64` scale precision, doc-markdown cleanup,
    and `format_push_string` removal), validated with targeted nextest
    (`8 passed`, `0 failed`) and mandatory touched-crate clippy gate success.
261. Use `274-xiuxian-wendao-zhenfa-xml-lite-top-level-test-boundary-convergence-2026-03-02.md`
    for replacing `xiuxian-wendao` zhenfa XML-Lite source-include harness with
    package-top integration tests through a stable public adapter, deleting
    obsolete `tests/unit` include artifacts, and validating with targeted
    nextest (`5 passed`, `0 failed`) plus mandatory touched-crate clippy gate
    success (`--features zhenfa-router`).
262. Use `275-xiuxian-llm-vision-scrub-pedantic-warning-cleanup-2026-03-02.md`
    for closing `xiuxian-llm` vision scrub pedantic warnings
    (`manual_contains` and `redundant_closure_for_method_calls`) with
    root-cause code fixes and validation via targeted nextest
    (`10 passed`, `0 failed`) plus mandatory touched-crate clippy gate success.
263. Use `276-xiuxian-vector-search-impl-test-remap-elimination-via-test-support-boundary-2026-03-02.md`
    for removing `xiuxian-vector` `search_impl` source-include remaps by adding a
    stable `test_support` boundary, keeping test behavior unchanged, and
    validating via targeted nextest (`6 passed`, `0 failed`) plus mandatory
    touched-crate clippy gate success.
264. Use `277-xiuxian-daochang-memory-decay-recall-credit-feedback-test-remap-reduction-wave-2026-03-02.md`
    for removing three `xiuxian-daochang` memory-lane source includes
    (`decay/recall_credit/memory_recall_feedback`) by migrating tests to
    stable `test_support` wrappers, deleting obsolete include-driven test
    fragments, and validating with targeted nextest (`16 passed`, `0 failed`)
    plus mandatory touched-crate clippy gate success.
265. Use `278-xiuxian-daochang-admission-test-remap-elimination-via-test-support-2026-03-02.md`
    for removing `agent_admission` source-include remap by introducing an
    admission test-support adapter layer and migrating tests to stable
    `xiuxian_daochang::test_support` contracts, validated with targeted nextest
    (`24 passed`, `0 failed`) plus mandatory touched-crate clippy gate success.
266. Use `279-xiuxian-daochang-gateway-runtime-and-llm-proxy-test-remap-elimination-2026-03-02.md`
    for removing gateway runtime and LLM-proxy source include remaps by adding
    stable gateway test-support wrappers, deleting obsolete include fragments,
    and validating with targeted nextest (`17 passed`, `0 failed`) plus
    mandatory touched-crate clippy gate success.
267. Use `280-xiuxian-daochang-mcp-startup-test-remap-elimination-via-test-support-2026-03-02.md`
    for removing `agent_mcp_startup` source include remap through a stable
    `test_support::startup_connect_config` boundary, while preserving strict vs
    non-strict startup policy assertions and validating with targeted nextest
    (`27 passed`, `0 failed`) plus mandatory touched-crate clippy gate success.
268. Use `281-xiuxian-daochang-managed-runtime-redis-metrics-test-remap-elimination-wave-2026-03-03.md`
    for removing three additional `xiuxian-daochang` source-remap lanes
    (`channels_managed_runtime_unit`, `session_redis_backend_unit`,
    `agent_memory_recall_metrics`) via new `test_support` adapters, replacing
    include-driven harnesses with package-top test entrypoints, and validating
    with targeted nextest (`15 passed`, `0 failed`) plus mandatory
    touched-crate clippy gate success.
269. Use `282-xiuxian-daochang-nodes-warmup-remap-elimination-via-shared-warmup-options-module-2026-03-03.md`
    for removing `nodes_warmup` source-remap harness by extracting warmup
    option resolution into a shared library module, reusing it in both runtime
    node code and integration tests through `test_support`, and validating
    with targeted nextest (`18 passed`, `0 failed`) plus mandatory
    touched-crate clippy gate success.
270. Use `283-xiuxian-daochang-memory-stream-consumer-remap-elimination-via-test-support-boundary-2026-03-03.md`
    for removing `agent_memory_stream_consumer_unit` source remaps through
    stable test-support wrappers over stream parsing/read/ack/promotion lanes,
    replacing include-driven harnesses with package-top tests, and validating
    with targeted nextest (`32 passed`, `4 skipped`, `0 failed`) plus
    mandatory touched-crate clippy gate success.
271. Use `284-xiuxian-daochang-zhenfa-test-remap-elimination-via-test-support-adapters-2026-03-03.md`
    for removing `agent_zhenfa_unit` source remap via stable zhenfa
    test-support wrappers (bridge + valkey hooks + reward sink adapters),
    with targeted nextest validation (`47 passed`, `5 skipped`, `0 failed`)
    and mandatory touched-crate clippy gate success.
272. Use `285-xiuxian-daochang-session-context-test-remap-elimination-via-test-support-window-ops-adapters-2026-03-03.md`
    for removing `agent_session_context_unit` source remap by exposing
    minimal crate-internal session-context test helpers and wrapping them in
    `test_support`, validated with targeted nextest (`49 passed`, `5 skipped`,
    `0 failed`) and mandatory touched-crate clippy gate success.
273. Use `286-xiuxian-daochang-memory-recall-state-test-remap-elimination-via-test-support-payload-adapter-2026-03-03.md`
    for removing `agent_memory_recall_state_unit` source remap by adding a
    stable memory-recall-state payload adapter in `test_support`, migrating
    persistence-compat tests off direct session writes, and validating with
    targeted nextest (`4 passed`, `2 skipped`, `0 failed`) plus mandatory
    touched-crate clippy gate success.
274. Use `287-xiuxian-daochang-bootstrap-test-remap-elimination-via-bootstrap-test-support-layer-2026-03-03.md`
    for removing bootstrap source includes by introducing a dedicated
    bootstrap `test_support` layer, converging tests to standard package-top
    entrypoints, and validating with targeted nextest (`22 passed`,
    `2 skipped`, `0 failed`) plus mandatory touched-crate clippy gate success.
275. Use `288-xiuxian-daochang-embedding-test-remap-elimination-via-test-support-adapters-2026-03-03.md`
    for removing four `embedding` source includes by introducing stable
    embedding test-support adapters and converting the harness to package-top
    `#[path]` modules, validated with targeted nextest (`8 passed`, `0 failed`)
    plus mandatory touched-crate clippy gate success.
276. Use `289-xiuxian-daochang-discord-runtime-test-remap-elimination-via-test-support-runtime-bridge-2026-03-03.md`
    for removing Discord runtime source includes by adding crate-internal
    runtime test bridges and public `test_support` wrappers, converting the
    top-level harness to package-top `#[path]` modules, and validating with
    targeted nextest (`36 passed`, `0 failed`) plus mandatory touched-crate
    clippy gate success.
277. Use `290-xiuxian-daochang-telegram-runtime-test-remap-elimination-and-suite-zero-remap-convergence-2026-03-03.md`
    for removing the remaining Telegram runtime source includes by converging
    tests to package-top entrypoints over stable `test_support` wrappers,
    validating with targeted nextest (`110 passed`, `0 failed`) plus mandatory
    touched-crate clippy gate success, and closing `xiuxian-daochang/tests` remap
    debt to zero.
278. Use `291-xiuxian-daochang-mcp-dispatch-modularization-and-too-many-lines-convergence-2026-03-03.md`
    for decomposing the MCP dispatch chain into focused native/zhenfa/mcp
    stages, removing repeated telemetry blocks through shared logging helpers,
    closing the previous `mcp.rs` `too_many_lines` warning by root-cause
    structural fixes, and validating with targeted nextest (`15 passed`,
    `0 failed`) plus mandatory touched-crate clippy gate success (`0` warnings).
279. Use `292-xiuxian-daochang-llm-message-integrity-inline-test-migration-to-package-top-tests-2026-03-03.md`
    for migrating LLM message-integrity tests out of source-inline `#[cfg(test)]`
    modules into package-top `tests/llm` through stable `test_support` and
    `test_api` boundaries, validating with targeted nextest (`28 passed`,
    `0 failed`) and touched-crate clippy gate success.
280. Use `293-rust-inline-test-zero-convergence-xiuxian-config-core-and-xiuxian-llm-2026-03-03.md`
    for removing the remaining source-inline `#[cfg(test)]` hooks in
    `xiuxian-config-core` and `xiuxian-llm` by migrating them to package-top
    `tests/` binaries via hidden test-support boundaries, validating with
    targeted nextest (`12 passed` and `9 passed`) and touched-crate clippy
    gate success for both crates.
281. Use `294-xiuxian-llm-provider-module-allow-dead-code-elimination-via-feature-gating-2026-03-03.md`
    for replacing provider-module `allow(dead_code)` suppressions with explicit
    `provider-litellm` feature gating in `xiuxian-llm`, validated by targeted
    nextest (`9 passed`, `1 skipped`, `0 failed`) and touched-crate clippy
    gate success.
282. Use `295-rust-allow-dead-code-zero-convergence-via-feature-accurate-apis-2026-03-03.md`
    for closing the final `allow(dead_code)` suppression debt across Rust
    crates by combining provider-level feature gating (`xiuxian-llm`) and
    feature-specific constructor structure (`xiuxian-qianji`), validated with
    targeted nextest and touched-crate clippy gate success.
283. Use `296-xiuxian-daochang-litellm-compat-anthropic-pipeline-modularization-2026-03-03.md`
    for splitting `xiuxian-daochang` `litellm` compatibility into focused runtime
    orchestration and Anthropic custom-base conversion modules, validated with
    targeted nextest (`38 passed`, `0 failed`) and touched-crate clippy gate
    success.
284. Use `297-xiuxian-qianji-compiler-feature-split-removes-unused-self-suppression-2026-03-03.md`
    for removing `xiuxian-qianji` compiler `unused_self` suppression via
    cfg-specific mechanism builder variants and cfg-aware dispatch calls,
    validated with targeted nextest (`2 passed`, `0 failed`) and touched-crate
    clippy gate success.
285. Use `298-xiuxian-qianji-compiler-formal-audit-module-extraction-2026-03-03.md`
    for extracting formal-audit helper logic from `xiuxian-qianji` compiler
    into a dedicated submodule while preserving behavior, validated with
    targeted nextest (`2 passed`, `0 failed`), touched-crate clippy gate
    success, and global Rust suppression count staying at zero.
286. Use `299-xiuxian-qianji-compiler-llm-node-module-extraction-2026-03-03.md`
    for extracting `xiuxian-qianji` compiler node-level LLM endpoint/config
    parsing into `compiler/llm_node.rs`, reducing compiler core density and
    validating with targeted nextest (`2 passed`, `0 failed`) plus touched-crate
    clippy gate success.
287. Use `300-xiuxian-qianji-compiler-wendao-refresh-module-extraction-2026-03-03.md`
    for extracting `xiuxian-qianji` compiler `wendao_refresh` parameter parsing
    into a dedicated module, reducing compiler core density while preserving
    defaults and validating with targeted nextest (`2 passed`, `0 failed`) plus
    touched-crate clippy gate success.
288. Use `301-xiuxian-daochang-discord-listen-runtime-bridge-and-skeleton-removal-2026-03-03.md`
    for replacing Discord `Channel::listen` hardcoded `not implemented` failure
    with a real gateway runtime bridge, adding deterministic token validation,
    updating stale skeleton docs, and validating with targeted Discord nextest
    (`68 passed`, `0 failed`) plus touched-crate clippy gate execution evidence.
289. Use `302-xiuxian-qianji-compiler-io-mechanism-config-module-extraction-2026-03-03.md`
    for extracting command/write_file/suspend parameter decoding from
    `xiuxian-qianji` compiler core into `compiler/io_mechanisms.rs`, reducing
    compiler core density while preserving defaults/fallbacks and validating
    with targeted nextest (`2 passed`, `0 failed`) plus touched-crate clippy
    gate success.
290. Use `303-xiuxian-qianji-compiler-security-scan-and-wendao-ingester-module-extraction-2026-03-03.md`
    for extracting `security_scan` and `wendao_ingester` parameter decoding from
    `xiuxian-qianji` compiler core into dedicated domain modules, reducing core
    density while preserving runtime-default behavior and validating with
    targeted nextest (`2 passed`, `0 failed`) plus touched-crate clippy gate
    success.
291. Use `304-xiuxian-qianji-compiler-annotation-and-affinity-module-extraction-2026-03-03.md`
    for extracting annotation binding parsing and execution-affinity resolution
    from `xiuxian-qianji` compiler core into `compiler/annotation.rs`, reducing
    orchestration density while preserving defaults and validating with targeted
    nextest (`2 passed`, `0 failed`) plus touched-crate clippy gate success.
292. Use `305-xiuxian-qianji-compiler-calibration-and-router-module-extraction-2026-03-03.md`
    for extracting calibration target parsing and router branch parsing/weight
    validation from `xiuxian-qianji` compiler core into dedicated modules,
    reducing orchestration density while preserving behavior and validating with
    targeted nextest (`2 passed`, `0 failed`) plus touched-crate clippy gate success.
293. Use `306-xiuxian-qianji-compiler-graph-assembly-module-extraction-2026-03-03.md`
    for extracting manifest graph node/edge assembly from `xiuxian-qianji`
    compiler core into `compiler/graph_assembly.rs`, preserving compile behavior
    while reducing orchestration density and validating with targeted nextest
    (`2 passed`, `0 failed`) plus touched-crate clippy gate success.
294. Use `307-xiuxian-qianji-compiler-task-type-typed-dispatch-2026-03-03.md`
    for replacing string-literal task dispatch in `xiuxian-qianji` compiler
    core with typed `TaskType` parsing, centralizing unknown-task validation
    while preserving task mapping and validating with targeted nextest
    (`2 passed`, `0 failed`) plus touched-crate clippy gate success.
295. Use `308-xiuxian-qianji-compiler-cfg-dispatch-shell-convergence-2026-03-03.md`
    for converging feature-gated `formal_audit`/`llm` dispatch into thin method
    shells, removing inline cfg noise from `build_mechanism` while preserving
    behavior and validating with targeted nextest (`2 passed`, `0 failed`) plus
    touched-crate clippy gate success.
296. Use `309-xiuxian-qianji-compiler-leaf-task-mechanism-builders-extraction-2026-03-03.md`
    for extracting non-`self` dependent leaf task mechanism constructors from
    `xiuxian-qianji` compiler core into `compiler/task_mechanisms.rs`, reducing
    orchestration density while preserving behavior and validating with targeted
    nextest (`2 passed`, `0 failed`) plus touched-crate clippy gate success.
297. Use `310-xiuxian-qianji-compiler-stateful-lanes-module-extraction-2026-03-03.md`
    for extracting stateful mechanism construction lanes (`annotation`,
    `formal_audit`, `llm`) from `xiuxian-qianji` compiler core into
    `compiler/stateful_mechanisms.rs`, preserving feature-gated behavior while
    reducing orchestration density and validating with targeted nextest
    (`2 passed`, `0 failed`) plus touched-crate clippy gate success.
298. Use `311-xiuxian-qianji-compiler-llm-client-resolution-module-extraction-2026-03-03.md`
    for extracting node-level/global LLM client resolution and transport client
    construction from `xiuxian-qianji` compiler core into
    `compiler/llm_client.rs`, reducing orchestration density while preserving
    behavior and validating with targeted nextest (`2 passed`, `0 failed`) plus
    touched-crate clippy gate success.
299. Use `312-xiuxian-qianji-compiler-manifest-and-topology-guard-extraction-2026-03-03.md`
    for extracting TOML manifest parsing and static cycle guard from
    `xiuxian-qianji` compiler core into `compiler/manifest_parser.rs` and
    `compiler/topology_validation.rs`, reducing orchestration density while
    preserving behavior and validating with targeted nextest (`2 passed`,
    `0 failed`) plus touched-crate clippy gate success.
300. Use `313-xiuxian-qianji-compiler-mechanism-dispatch-module-extraction-2026-03-03.md`
    for extracting task-type mechanism dispatch from `xiuxian-qianji` compiler
    core into `compiler/mechanism_dispatch.rs`, reducing orchestration density
    while preserving behavior and validating with targeted nextest (`2 passed`,
    `0 failed`) plus touched-crate clippy gate success.
301. Use `314-xiuxian-qianji-compiler-cfg-dispatch-relocation-to-mechanism-dispatch-2026-03-03.md`
    for relocating remaining cfg-heavy `formal_audit` and `llm` dispatch plus
    knowledge/annotation mechanism build paths from `xiuxian-qianji` compiler
    core into `compiler/mechanism_dispatch.rs`, reducing compiler shell density
    while preserving behavior and validating with targeted nextest (`2 passed`,
    `0 failed`) plus touched-crate clippy gate success.
302. Use `315-xiuxian-qianji-mechanism-dispatch-submodule-split-stateless-stateful-cfg-2026-03-03.md`
    for splitting `xiuxian-qianji` mechanism dispatch internals into
    `compiler/mechanism_dispatch/stateless.rs` and
    `compiler/mechanism_dispatch/stateful_cfg.rs`, isolating cfg-sensitive
    routing from stateless paths while preserving behavior and validating with
    targeted nextest (`2 passed`, `0 failed`) plus touched-crate clippy gate
    success.
303. Use `316-xiuxian-qianji-mechanism-dispatch-leaf-routing-module-extraction-2026-03-03.md`
    for extracting non-stateful leaf-task routing into
    `compiler/mechanism_dispatch/leaf_dispatch.rs` and converging dispatch into
    three layers (`stateless`, `stateful_cfg`, `leaf`), preserving behavior and
    validating with targeted nextest (`2 passed`, `0 failed`) plus touched-crate
    clippy gate success.
304. Use `317-xiuxian-qianji-stateful-cfg-directory-module-split-formal-audit-llm-2026-03-03.md`
    for converting `compiler/mechanism_dispatch/stateful_cfg.rs` into a
    directory module (`stateful_cfg/mod.rs`) and splitting cfg-sensitive routing
    into `stateful_cfg/formal_audit.rs` and `stateful_cfg/llm.rs`, preserving
    behavior and validating with targeted nextest (`2 passed`, `0 failed`) plus
    touched-crate clippy gate success.
305. Use `318-xiuxian-qianji-leaf-dispatch-directory-module-split-io-quality-wendao-router-2026-03-03.md`
    for converting `compiler/mechanism_dispatch/leaf_dispatch.rs` into a
    directory module and splitting leaf routing into `io_control`,
    `quality_guard`, and `wendao_router` domains while preserving behavior and
    validating with targeted nextest (`2 passed`, `0 failed`) plus touched-crate
    clippy gate success.
306. Use `319-xiuxian-qianji-task-mechanisms-directory-module-split-io-quality-wendao-router-2026-03-04.md`
    for converting `compiler/task_mechanisms.rs` into a directory module and
    splitting mechanism constructors into `task_mechanisms/io_control.rs`,
    `task_mechanisms/quality.rs`, and `task_mechanisms/wendao_router.rs` while
    preserving behavior and validating with targeted nextest (`2 passed`,
    `0 failed`) plus touched-crate clippy gate success.
307. Use `320-xiuxian-qianji-task-mechanisms-interface-only-reexport-via-compiler-scope-2026-03-04.md`
    for converting `task_mechanisms/mod.rs` into an interface-only re-export
    layer by applying `pub(in crate::engine::compiler)` visibility in child
    modules and removing forwarding wrappers, while preserving behavior and
    validating with targeted nextest (`2 passed`, `0 failed`) plus touched-crate
    clippy gate success.
308. Use `321-xiuxian-qianji-mechanism-dispatch-unified-resolver-chain-interface-2026-03-04.md`
    for introducing a unified resolver-chain interface in
    `compiler/mechanism_dispatch/resolver_chain.rs` and migrating top-level +
    leaf-level dispatch to that chain pattern, preserving behavior and
    validating with targeted nextest (`2 passed`, `0 failed`) plus touched-crate
    clippy gate success.
309. Use `322-xiuxian-qianji-mechanism-dispatch-context-based-resolver-signature-unification-2026-03-04.md`
    for unifying resolver signatures around a shared `DispatchContext` in
    `compiler/mechanism_dispatch/resolver_chain.rs`, reducing repeated
    `(task_type, compiler, node_def)` parameter passing while preserving
    behavior and validating with targeted nextest (`2 passed`, `0 failed`) plus
    touched-crate clippy gate success.
310. Use `323-xiuxian-qianji-const-resolver-pipelines-for-root-and-leaf-dispatch-2026-03-04.md`
    for replacing inline resolver arrays with fixed `const` resolver pipelines
    in both root and leaf dispatch modules, preserving behavior and validating
    with targeted nextest (`2 passed`, `0 failed`) plus touched-crate clippy
    gate success.
311. Use `324-xiuxian-qianji-compiler-dispatch-routes-integration-tests-2026-03-04.md`
    for adding route-focused compiler integration tests that lock stateless
    (`knowledge`) and leaf (`command`) dispatch success paths, non-`llm`
    formal-audit feature-guard behavior, and unknown-task topology errors,
    validated with targeted nextest (`6 passed`, `0 failed`) plus touched-crate
    clippy gate success.
312. Use `325-xiuxian-qianji-compiler-directory-module-entry-migration-2026-03-04.md`
    for migrating `xiuxian-qianji` compiler entry from `compiler.rs` to
    `compiler/mod.rs`, aligning module layout with directory-module conventions
    while preserving public paths and validating with targeted nextest
    (`6 passed`, `0 failed`) plus touched-crate clippy gate success.
313. Use `326-xiuxian-qianji-leaf-dispatch-route-coverage-expansion-2026-03-04.md`
    for expanding compiler integration coverage of leaf dispatch routes
    (`write_file`, `suspend`, `router`, `wendao_ingester`,
    `wendao_refresh`) and adding router invalid-weight error coverage, validated
    with targeted nextest (`12 passed`, `0 failed`) plus touched-crate clippy
    gate success.
314. Use `327-xiuxian-qianji-quality-guard-dispatch-coverage-expansion-2026-03-04.md`
    for completing compiler route coverage of `quality_guard` leaf dispatch
    lanes (`calibration`, `mock`, `security_scan`), validated with targeted
    nextest (`15 passed`, `0 failed`) plus touched-crate clippy gate success.
315. Use `328-xiuxian-qianji-annotation-affinity-dispatch-contract-tests-2026-03-04.md`
    for adding annotation dispatch contract tests that validate explicit
    execution-affinity mapping (`agent_id`, `role_class`) and persona-based
    role-class derivation, validated with targeted nextest (`17 passed`,
    `0 failed`) plus touched-crate clippy gate success.
316. Use `329-xiuxian-qianji-stateful-dispatch-and-llm-guard-coverage-2026-03-04.md`
    for adding stateful dispatch route coverage of native `formal_audit`
    compilation and non-feature `llm` task guard behavior, validated with
    targeted nextest (`19 passed`, `0 failed`) plus touched-crate clippy gate
    success.
317. Use `330-xiuxian-qianji-llm-feature-dispatch-route-tests-and-cfg-guard-fix-2026-03-04.md`
    for adding dedicated `--features llm` compiler dispatch route tests
    (positive `llm`/augmented `formal_audit` paths and missing-client failure)
    and fixing feature-specific dead code via precise cfg scoping, validated
    with targeted nextest (`19 passed` default lane + `3 passed` llm lane) and
    touched-crate clippy gate success.
318. Use `331-xiuxian-llm-deepseek-warning-convergence-and-config-contract-alignment-2026-03-04.md`
    for converging deepseek OCR warnings in `xiuxian-llm` without suppression
    (doc markdown, default-init reassignment, line-budget modularization,
    pedantic follow-up cleanup) and aligning deepseek config test expectations
    with embedded defaults (`max_new_tokens = 64`), validated by strict clippy
    plus targeted nextest (`15 passed` for llm deepseek tests, `19 + 3 passed`
    for qianji dispatch lanes).
319. Use `332-xiuxian-llm-deepseek-engine-directory-modularization-and-config-default-contract-correction-2026-03-04.md`
    for completing deepseek native `engine` directory modularization
    (`cache_io`, `image_decode`, `retry`, `telemetry`), fixing post-split
    module import paths, and correcting deepseek config test expectations to
    match embedded defaults (`base_size = 1024`, `image_size = 640`,
    `max_new_tokens = 256`), validated by strict clippy plus targeted nextest
    (`15 passed` for llm deepseek tests, `19 + 3 passed` for qianji dispatch
    lanes).
320. Use `333-xiuxian-llm-deepseek-native-env-directory-modularization-2026-03-04.md`
    for splitting deepseek native env orchestration into domain modules
    (`env/device.rs`, `env/parse.rs`, `env/paths.rs`) with interface-only
    `env/mod.rs`, preserving sibling call paths and test hooks while enforcing
    scoped visibility and cfg-safe imports, validated by strict clippy plus
    targeted nextest (`15 passed` for llm deepseek tests, `19 + 3 passed` for
    qianji dispatch lanes).
321. Use `334-xiuxian-llm-deepseek-full-domain-directory-modularization-2026-03-04.md`
    for completing deepseek domain-wide directory modularization by converting
    `config`, `runtime`, `native/cache`, and `native/engine` to focused module
    trees (interface-only `mod.rs` with domain submodules), while preserving
    behavior and validating with strict clippy plus targeted nextest (`15
    passed` for llm deepseek tests, `19 + 3 passed` for qianji dispatch
    lanes).
322. Use `335-xiuxian-llm-deepseek-device-policy-probe-split-2026-03-04.md`
    for converting deepseek device selection from a single module into
    `env/device/mod.rs` + `env/device/policy.rs` + `env/device/probe.rs`,
    separating policy resolution from platform probing while preserving
    deepseek runtime behavior and validating with strict clippy plus targeted
    nextest (`15 passed` for llm deepseek tests, `19 + 3 passed` for qianji
    dispatch lanes).
323. Use `336-xiuxian-llm-deepseek-cache-key-tests-migrated-to-top-level-tests-2026-03-04.md`
    for moving deepseek cache-key tests from module-local `native/cache/tests`
    to crate-level `tests/llm_vision_deepseek_cache_key_unit.rs`, adding a
    test-support bridge with struct-based input to avoid argument-count
    warnings, and validating with strict clippy plus targeted nextest (`17
    passed` for llm deepseek tests, `19 + 3 passed` for qianji dispatch
    lanes).
324. Use `337-xiuxian-llm-deepseek-facade-thinning-and-inference-lane-split-2026-03-04.md`
    for thinning deepseek facade surface by extracting test-only bridge APIs
    into `deepseek/test_api.rs` and converting `inference.rs` into an
    interface+lane directory module (`inference/mod.rs`,
    `inference/runtime_lane.rs`), validated with strict clippy plus targeted
    nextest (`17 passed` for llm deepseek tests, `19 + 3 passed` for qianji
    dispatch lanes).
325. Use `338-xiuxian-llm-deepseek-valkey-init-ops-split-and-failure-path-tests-2026-03-04.md`
    for splitting deepseek valkey cache into init/ops layers
    (`valkey/client_init.rs`, `valkey/ops.rs`), adding explicit test hooks for
    invalid-url/connection-failure/timeout-clamp paths, and validating with
    strict clippy plus targeted nextest (`20 passed` for llm deepseek tests,
    `19 + 3 passed` for qianji dispatch lanes).
326. Use `339-xiuxian-llm-deepseek-test-api-domain-split-2026-03-04.md`
    for converting deepseek test bridge `test_api.rs` into domain modules
    (`test_api/config_runtime.rs`, `test_api/native_cache.rs`,
    `test_api/native_device.rs`) with interface-only `test_api/mod.rs` while
    preserving helper signatures and validating with strict clippy plus
    targeted nextest (`20 passed` for llm deepseek tests, `19 + 3 passed` for
    qianji dispatch lanes).
327. Use `340-xiuxian-llm-deepseek-local-cache-policy-storage-split-and-top-level-policy-tests-2026-03-04.md`
    for completing deepseek local-cache modularization by replacing legacy
    `local.rs` with focused `local/policy.rs` + `local/storage.rs`, wiring
    local-cache test hooks through the deepseek test API and crate
    `test_support`, and adding crate-top policy tests for eviction and zero-cap
    normalization behavior, validated with strict clippy plus targeted nextest
    (`22 passed` for llm deepseek tests, `19 + 3 passed` for qianji dispatch
    lanes).
328. Use `341-xiuxian-llm-deepseek-local-cache-shared-read-path-and-cache-io-normalization-2026-03-04.md`
    for introducing a shared-string (`Arc<str>`) local-cache read path and
    cache-io normalization that avoids avoidable re-allocation in local/valkey
    cache-hit paths while preserving telemetry + backfill semantics, with
    touched-crate pedantic cleanup in deepseek image decode env parsing and
    validation via strict clippy plus targeted nextest (`22 passed` for llm
    deepseek tests, `19 + 3 passed` for qianji dispatch lanes).
329. Use `342-xiuxian-llm-deepseek-cache-text-domain-extraction-and-normalization-contract-tests-2026-03-04.md`
    for extracting deepseek cache-text normalization into a dedicated
    `native/cache/text.rs` domain module, migrating `cache_io` to consume that
    module, exposing normalization test hooks through deepseek test bridges, and
    adding crate-top contract tests (including no-reallocation owned fast-path
    assertion), validated via strict clippy plus targeted nextest (`25 passed`
    for llm deepseek tests, `19 + 3 passed` for qianji dispatch lanes).
330. Use `343-xiuxian-llm-deepseek-engine-cache-io-read-write-split-and-write-contract-tests-2026-03-04.md`
    for converting deepseek engine cache I/O from a single `cache_io.rs` into a
    directory module split (`cache_io/read.rs` + `cache_io/write.rs` +
    interface-only `cache_io/mod.rs`), exposing a dedicated write-path test hook
    through deepseek test bridges, and adding crate-top write contract tests
    (empty payload skip + non-empty payload store), validated via strict clippy
    plus targeted nextest (`27 passed` for llm deepseek tests, `19 + 3 passed`
    for qianji dispatch lanes).
331. Use `344-xiuxian-llm-deepseek-cache-layer-typed-dispatch-in-cache-read-path-2026-03-04.md`
    for replacing deepseek cache read string dispatch (`\"local\"`/`\"valkey\"`)
    with a typed `CacheLayer` enum in `cache_io/read.rs`, updating engine
    call-sites to variant-based dispatch, and preserving telemetry label output
    through enum mapping, validated via strict clippy plus targeted nextest
    (`27 passed` for llm deepseek tests, `19 + 3 passed` for qianji dispatch
    lanes).
332. Use `345-xiuxian-llm-deepseek-cache-layer-module-extraction-and-label-contract-test-2026-03-04.md`
    for extracting `CacheLayer` from `cache_io/read.rs` into dedicated
    `cache_io/layer.rs`, wiring enum-label test hooks through deepseek native
    and test-api bridges, and adding crate-top contract coverage that locks
    `Local/Valkey` telemetry labels, validated via strict clippy plus targeted
    nextest (`28 passed` for llm deepseek tests, `19 + 3 passed` for qianji
    dispatch lanes).
333. Use `346-xiuxian-llm-deepseek-native-cache-test-facade-consolidation-2026-03-04.md`
    for consolidating deepseek native cache-test forwarding into
    `DeepseekNativeCacheTestFacade` (`native/test_cache.rs`), removing 11
    standalone forwarders from `native/mod.rs`, and routing `test_api` cache
    helpers through the façade while keeping top-level test-support contracts
    unchanged, validated via strict clippy plus targeted nextest (`28 passed`
    for llm deepseek tests, `19 + 3 passed` for qianji dispatch lanes).
334. Use `347-xiuxian-llm-test-support-deepseek-cache-facade-and-legacy-wrapper-compat-2026-03-04.md`
    for introducing `DeepseekCacheTestFacade` in `src/test_support.rs` as the
    single implementation owner for deepseek cache test helpers, while keeping
    all existing free-function helper signatures as compatibility wrappers so
    downstream tests remain unchanged, validated via strict clippy plus targeted
    nextest (`28 passed` for llm deepseek tests, `19 + 3 passed` for qianji
    dispatch lanes).
335. Use `348-xiuxian-llm-test-support-deepseek-cache-subfacades-and-wrapper-parity-contracts-2026-03-04.md`
    for splitting deepseek cache test-support internals into domain modules
    (`key`, `valkey`, `local`, `text`, `write`, delegating `facade`) under
    `src/test_support/deepseek_cache/`, keeping legacy wrapper signatures in
    `src/test_support.rs`, and adding crate-top wrapper/facade parity contract
    tests, validated via strict clippy plus targeted nextest (`33 passed` for
    llm deepseek tests, `19 + 3 passed` for qianji dispatch lanes).
336. Use `349-xiuxian-llm-test-support-directory-moduleization-and-interface-only-mod-2026-03-04.md`
    for converting `test_support` from `src/test_support.rs` to directory
    module layout (`src/test_support/mod.rs` + focused
    `acceleration`/`deepseek_config`/`deepseek_runtime`/`deepseek_cache_api`
    submodules), keeping public helper signatures stable and preserving cache
    wrapper-to-facade routing, validated via strict clippy plus targeted
    nextest (`33 passed` for llm deepseek tests, `19 + 3 passed` for qianji
    dispatch lanes).
337. Use `350-xiuxian-config-core-resolve-directory-moduleization-and-domain-split-2026-03-04.md`
    for replacing monolithic `src/resolve.rs` with a domain-split directory
    module (`resolve/mod.rs` + `discover/io/merge/namespace`), preserving
    public resolver APIs while isolating concerns (path discovery, TOML IO,
    merge semantics, namespace extraction), validated via strict clippy and
    full crate nextest (`12 passed`).
338. Use `351-xiuxian-wendao-skill-vfs-resolver-directory-moduleization-and-concern-split-2026-03-04.md`
    for converting `skill_vfs::resolver` from a monolithic file into
    concern-split directory modules (`core`/`mount`/`resolve_uri`/`read`) with
    `resolver/mod.rs` entrypoint re-exporting `SkillVfsResolver`, preserving
    resolver behavior and cache semantics, validated via strict clippy and
    targeted skill-vfs nextest (`13 passed`).
339. Use `352-xiuxian-wendao-skill-vfs-index-directory-moduleization-and-build-preload-semantic-split-2026-03-04.md`
    for converting `skill_vfs::index` from a monolithic file into focused
    directory modules (`mod`/`build`/`preload`/`semantic`), preserving
    `SkillNamespaceIndex` public query APIs and URI-key semantics while
    separating scan orchestration, preload indexing, and semantic extraction,
    validated via strict clippy and targeted skill-vfs nextest (`13 passed`).
340. Use `353-xiuxian-wendao-skill-vfs-asset-request-directory-moduleization-and-api-stability-2026-03-04.md`
    for converting `skill_vfs::asset_request` into focused directory modules
    (`types`/`build`/`normalize`/`read` with interface `mod.rs`), preserving
    public `WendaoAssetHandle`/`AssetRequest` API contracts and stripped-body
    cache behavior, validated via strict clippy and targeted asset+skill-vfs
    nextest (`19 passed`).
341. Use `354-xiuxian-qianji-swarm-discovery-directory-moduleization-and-registry-concern-split-2026-03-04.md`
    for converting monolithic `swarm::discovery` into concern-split directory
    modules (`model`/`registry`/`parse`/`util` with interface `mod.rs`),
    preserving `GlobalSwarmRegistry` and node model APIs while isolating
    heartbeat/discovery orchestration from record parsing/utilities, validated
    via strict clippy plus swarm discovery and compiler-dispatch nextest lanes
    (`2 + 19 + 3 passed`).
342. Use `355-xiuxian-qianji-swarm-possession-directory-moduleization-and-transport-concern-split-2026-03-04.md`
    for converting monolithic `swarm::possession` into concern-split directory
    modules (`model`/`bus`/`error_map`/`util` with interface `mod.rs`),
    preserving remote possession protocol and transport APIs while separating
    request/response models from Valkey bus orchestration and error-response
    mapping, validated via strict clippy plus qianji dispatch/swarm lanes
    (`19 + 3 + 2 passed`).
343. Use `356-xiuxian-qianji-swarm-worker-runtime-directory-moduleization-and-execution-scheduler-split-2026-03-04.md`
    for converting `swarm::engine::worker::runtime` into concern-split
    directory modules (`execution`/`scheduler`/`reporting`/`telemetry` with
    interface `mod.rs`), preserving worker run-loop and scheduler-wiring
    behavior while isolating responsibilities, validated via strict clippy plus
    qianji dispatch/swarm lanes (`19 + 3 + 2 passed`).
344. Use `357-xiuxian-qianji-swarm-engine-types-directory-moduleization-and-model-boundary-split-2026-03-04.md`
    for converting mixed-responsibility `swarm::engine::types` into focused
    directory modules (`runtime`/`agent`/`options`/`report` with interface
    `mod.rs`), preserving public swarm type APIs while isolating internal
    runtime-only structures behind `pub(in crate::swarm::engine)` boundaries,
    validated via strict clippy plus qianji dispatch/swarm lanes
    (`19 + 3 + 2 passed`).
345. Use `358-xiuxian-qianji-bootcamp-directory-moduleization-and-workflow-runtime-llm-split-2026-03-04.md`
    for replacing monolithic `bootcamp.rs` with a concern-split directory
    module (`mod`/`model`/`workflow`/`manifest`/`runtime`/`llm`), preserving
    public bootcamp APIs (`run_workflow`, `run_workflow_with_mounts`,
    `run_scenario`, option/report types) while isolating run pipeline wiring,
    manifest resolution, runtime bootstrap, and feature-gated LLM client
    assembly, validated via strict clippy and targeted qianji nextest lanes
    (`3 + 19 + 3 passed`).
346. Use `359-xiuxian-qianji-consensus-manager-directory-moduleization-and-vote-connection-split-2026-03-04.md`
    for replacing monolithic `consensus/manager.rs` with a concern-split
    directory module (`mod`/`keys`/`time`/`connection`/`voting`), preserving
    public `ConsensusManager` APIs while separating vote key modeling,
    connection/reconnect command path, and quorum/winner resolution logic,
    validated via strict clippy and targeted qianji nextest lanes
    (`3 + 19 + 3 passed`).
347. Use `360-xiuxian-qianji-runtime-config-directory-moduleization-and-deterministic-env-override-semantics-2026-03-04.md`
    for replacing monolithic `runtime_config.rs` with concern-split directory
    modules (`constants`/`model`/`toml_config`/`env_vars`/`pathing`/`loader`/`resolve`),
    preserving public runtime-config APIs while adding deterministic
    `runtime_env.extra_env` override-state handling to prevent ambient env
    leakage in config-resolution tests, validated via strict clippy and
    targeted qianji nextest lanes (`9 + 19 + 3 passed`).
348. Use `361-xiuxian-qianji-executors-annotation-directory-moduleization-and-persona-markdown-boundary-split-2026-03-04.md`
    for replacing mixed-responsibility `executors/annotation.rs` with a
    concern-split directory module (`mod`/`context`/`persona_markdown`),
    preserving public `ContextAnnotator` API while separating execution flow
    from markdown-to-persona parsing utilities, validated via strict clippy and
    targeted qianji nextest lanes (`8 + 19 + 3 passed`).
349. Use `362-xiuxian-qianji-executors-formal-audit-directory-moduleization-and-native-llm-path-split-2026-03-04.md`
    for replacing mixed `executors/formal_audit.rs` with concern-split
    directory modules (`mod`/`native`/`llm`), preserving public formal-audit
    mechanism APIs while separating invariant-based native audit flow from
    llm-augmented critique control path, validated via strict clippy and
    targeted qianji nextest lanes (`3 + 7 + 19 passed`).
350. Use `363-xiuxian-qianji-scheduler-preflight-directory-moduleization-and-semantic-resolution-boundary-split-2026-03-04.md`
    for replacing mixed `scheduler/preflight.rs` with concern-split directory
    modules (`mod`/`context_path`/`mounts`/`wendao_uri`/`query`/`semantic`),
    preserving scheduler/preflight API contracts while separating runtime mount
    registry, context path parsing, URI dereference, query expansion, and
    semantic placeholder resolution policy, validated via strict clippy and
    targeted qianji nextest lanes (`8 + 19 + 3 passed`).
351. Use `364-xiuxian-qianji-executors-wendao-ingester-directory-moduleization-and-entity-persistence-boundary-split-2026-03-04.md`
    for replacing mixed `executors/wendao_ingester.rs` with concern-split
    directory modules (`mod`/`mechanism`/`entity`/`scope`/`persistence`),
    preserving public `WendaoIngesterMechanism` API while separating execution
    flow, entity/relation construction, graph scope resolution, and persistence
    side effects, validated via strict clippy and targeted qianji nextest lanes
    (`4 + 19 + 3 passed`).
352. Use `365-xiuxian-qianji-executors-wendao-refresh-directory-moduleization-and-input-refresh-boundary-split-2026-03-04.md`
    for replacing mixed `executors/wendao_refresh.rs` with concern-split
    directory modules (`mod`/`mechanism`/`input`/`refresh`), preserving public
    `WendaoRefreshMechanism` API while separating context input parsing and
    LinkGraph refresh execution strategy/fallback handling, validated via strict
    clippy and targeted qianji nextest lanes (`7 + 19 + 3 passed`).
353. Use `366-xiuxian-qianji-executors-llm-directory-moduleization-and-prompt-model-output-boundary-split-2026-03-04.md`
    for replacing mixed `executors/llm.rs` with concern-split directory modules
    (`mod`/`mechanism`/`model`/`prompt`/`output`), preserving public
    `LlmAnalyzer` API while separating prompt interpolation, model-selection
    policy, and output JSON/fallback shaping behavior, validated via strict
    clippy and targeted qianji nextest lanes (`10 + 19 + 3 passed`).
354. Use `367-xiuxian-qianji-executors-write-file-directory-moduleization-and-template-pathing-boundary-split-2026-03-04.md`
    for replacing mixed `executors/write_file.rs` with concern-split directory
    modules (`mod`/`mechanism`/`template`/`pathing`), preserving public
    `WriteFileMechanism` API while separating semantic/template interpolation
    and destination path security logic, validated via strict clippy and
    targeted qianji nextest lanes (`9 + 19 + 3 passed`).
355. Use `368-xiuxian-qianji-executors-security-scan-directory-moduleization-and-input-boundary-split-2026-03-04.md`
    for replacing mixed `executors/security_scan.rs` with concern-split
    directory modules (`mod`/`mechanism`/`input`), preserving public
    `SecurityScanMechanism` API while separating files-key/path input parsing
    from scanner execution and violation routing behavior, validated via strict
    clippy and core qianji nextest lanes (`19 + 3 passed`).
356. Use `369-xiuxian-qianji-swarm-possession-bus-directory-moduleization-and-valkey-transport-boundary-split-2026-03-04.md`
    for replacing mixed `swarm/possession/bus.rs` with concern-split directory
    modules (`mod`/`keys`/`connection`/`request`/`response`), preserving public
    `RemotePossessionBus` APIs while separating Valkey key modeling,
    connection/reconnect command path, request enqueue/claim logic, and
    response publish/wait flow, validated via strict clippy plus swarm/core
    qianji nextest lanes (`2 + 3 + 19 + 3 passed`).
357. Use `370-xiuxian-qianji-swarm-discovery-registry-directory-moduleization-and-connection-heartbeat-discovery-boundary-split-2026-03-04.md`
    for replacing mixed `swarm/discovery/registry.rs` with concern-split
    directory modules (`mod`/`keys`/`payload`/`connection`/`heartbeat`/`discover`),
    preserving public `GlobalSwarmRegistry` APIs while separating heartbeat
    payload shaping, Valkey connection lifecycle, heartbeat loop/write path,
    and discovery/candidate-selection flows, validated via strict clippy plus
    swarm/core qianji nextest lanes (`2 + 3 + 19 + 3 passed`).
358. Use `371-xiuxian-qianji-consensus-voting-directory-moduleization-and-vote-store-winner-timeout-boundary-split-2026-03-04.md`
    for replacing mixed `consensus/manager/voting.rs` with concern-split
    directory modules (`mod`/`submit`/`vote_store`/`winner`/`timeout`),
    preserving `ConsensusManager` external APIs while separating vote-submit
    orchestration, vote/output persistence and TTL refresh, winner lifecycle,
    and timeout checks with precise module-scoped visibility boundaries,
    validated via strict clippy plus consensus/swarm/core qianji nextest lanes
    (`3 + 5 + 19 + 3 passed`).
359. Use `372-xiuxian-qianji-python-module-directory-moduleization-and-pyo3-runtime-boundary-split-2026-03-04.md`
    for replacing mixed `python_module.rs` with concern-split directory modules
    (`mod`/`engine`/`scheduler`/`runtime`/`llm_bridge`), preserving Python API
    surface (`_xiuxian_qianji`, `QianjiEngine`, `QianjiScheduler`,
    `run_master_research_array`) while isolating runtime+JSON helpers and
    feature-gated LLM bridge logic, validated via default and feature-specific
    check/clippy lanes plus core qianji nextest lanes (`19 + 3 passed`).
360. Use `373-xiuxian-qianji-telemetry-valkey-directory-moduleization-and-publish-loop-boundary-split-2026-03-04.md`
    for replacing mixed `telemetry/valkey.rs` with concern-split directory
    modules (`mod`/`publish`), preserving public `ValkeyPulseEmitter` APIs
    while separating emitter-side queue/adaptive-sampling concerns from
    publisher-side connection/retry/backoff logic, validated via strict clippy
    plus telemetry/swarm/core qianji nextest lanes (`5 + 19 + 3 passed`).
361. Use `374-xiuxian-qianji-contracts-directory-module-boundary-split-for-execution-bindings-manifest-and-mechanism-2026-03-04.md`
    for converting mixed `contracts/mod.rs` into interface-only re-export
    module with domain splits (`execution`/`bindings`/`manifest`/`mechanism`),
    preserving serde aliases and all external contract paths while isolating
    schema and trait responsibilities, validated via strict clippy plus
    contracts-focused and core qianji nextest lanes (`9 + 19 + 3 passed`).
362. Use `375-xiuxian-qianji-scheduler-consensus-resolve-directory-moduleization-and-call-handler-policy-boundary-split-2026-03-04.md`
    for replacing mixed `scheduler/core/consensus/resolve.rs` with
    concern-split directory modules (`mod`/`call_ctx`/`handlers`/`policy`),
    preserving consensus decision and telemetry behavior while separating call
    context modeling, agreed/pending branch handlers, and target-progress
    policy logic, validated via strict clippy plus scheduler/core qianji
    nextest lanes (`5 + 19 + 3 passed`).
363. Use `376-xiuxian-qianji-scheduler-telemetry-directory-moduleization-and-alert-transition-boundary-split-2026-03-04.md`
    for replacing mixed `scheduler/core/telemetry.rs` with concern-split
    directory modules (`mod`/`node_transition`/`alerts`), preserving scheduler
    telemetry method signatures while separating non-blocking emission runtime,
    node transition event shaping, and alert/spike event builders, validated
    via strict clippy plus telemetry/scheduler/core qianji nextest lanes
    (`5 + 19 + 3 passed`).
364. Use `377-xiuxian-qianji-scheduler-types-directory-moduleization-and-runtime-bundle-boundary-split-2026-03-04.md`
    for replacing mixed `scheduler/core/types.rs` with concern-split directory
    modules (`mod`/`constants`/`consensus`/`remote`/`services`/`scheduler`/`constructors`),
    preserving scheduler core exports while separating runtime constants,
    checkpoint/outcome models, runtime service bundle, scheduler struct fields,
    and constructor family with explicit `pub(in crate::scheduler::core)`
    visibility boundaries; validated via strict clippy plus scheduler/core
    qianji lanes, with current `--features llm` lane externally blocked by
    `xiuxian-llm` compile errors.
365. Use `378-xiuxian-qianji-layout-unwrap-removal-and-llm-lane-revalidation-followup-2026-03-04.md`
    for follow-up unblock after `377`: replacing layout edge-routing
    `unwrap()` lookups with explicit safe branching, restoring
    `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines` hard-error
    pass and revalidating core + `--features llm` dispatch lanes
    (`19 + 3 passed`) in current workspace state.
366. Use `379-xiuxian-qianji-layout-pedantic-root-cause-cleanup-and-function-decomposition-2026-03-04.md`
    for the next layout convergence pass after `378`: replacing
    `push_str(&format!(...))` XML assembly with `write_fmt`-based output,
    completing missing public docs in layout APIs, and decomposing
    `compute_from_engine` into focused helpers to remove
    `clippy::too_many_lines` without suppression; validated with strict
    clippy and both core + `--features llm` dispatch lanes (`19 + 3 passed`).
367. Use `380-xiuxian-qianji-dng-layout-hardening-and-pedantic-convergence-2026-03-05.md`
    for DNG-mode follow-up convergence after `379`: preserving zone-aware BPMN
    and Obsidian graph semantics while removing `expect` panic paths, splitting
    long XML serialization into helper pipelines, and converging
    `xiuxian-qianji` `--all-targets --features llm` clippy gates with strict
    policy (core regression `20 passed`, LLM lane `3 passed`).
368. Use `381-xiuxian-qianji-context-uri-layout-convergence-on-latest-branch-2026-03-05.md`
    for latest-branch convergence on the `context_uri` layout line: preserving
    Wendao protocol metadata while removing panic lookups and allocation-heavy
    XML formatting, restoring probability/label rendering contracts, and
    validating strict touched-crate clippy plus core + LLM regression lanes
    (`20 + 3 passed`).
369. Use `382-xiuxian-qianji-layout-engine-directory-moduleization-on-context-uri-branch-2026-03-05.md`
    for replacing monolithic `layout/engine.rs` with concern-split directory
    modules (`mod`/`types`/`layout_core`/`deep_graph`) while preserving
    `context_uri` protocol metadata and deep-graph export behavior, validated
    via strict clippy and both core + LLM nextest lanes (`20 + 3 passed`).
370. Use `383-xiuxian-qianji-runtime-config-resolve-directory-moduleization-convergence-2026-03-05.md`
    for completing `runtime_config::resolve` directory-module migration by
    removing legacy `resolve.rs`, preserving public runtime-config APIs, and
    resolving `unnecessary_wraps` at root-cause level (no suppression
    attributes), validated via `check`, strict clippy, and targeted qianji
    runtime/dispatch regression lanes (`28 + 3 passed`).
371. Use `384-xiuxian-qianji-app-factory-directory-moduleization-and-modrs-interface-only-convergence-2026-03-05.md`
    for extracting `QianjiApp` and built-in pipeline presets from `lib.rs`
    into a dedicated `app/` directory module (`build`/`presets`/`qianji_app`)
    while restoring the `mod.rs` interface-only rule (declarations +
    re-exports only), validated via strict clippy and targeted qianji
    regression lanes (`31 + 5 passed`).
372. Use `385-xiuxian-types-rename-and-cross-crate-type-sink-audit-2026-03-05.md`
    for completing workspace-level rename from `omni-types` to
    `xiuxian-types`, sinking shared `MemoryGate*` contracts into the new
    canonical type crate, and producing a prioritized cross-crate type-sink
    backlog (`ToolSearchResult`/`HybridSearchResult`, knowledge-entry family,
    symbol taxonomy) with validated lint/test evidence.
373. Use `386-xiuxian-vector-tool-hybrid-result-sink-to-xiuxian-types-via-vector-contracts-2026-03-05.md`
    for sinking duplicated `xiuxian-vector` tool/hybrid result structs into
    `xiuxian-types` via canonical `VectorToolSearchResult` and
    `VectorHybridSearchResult` contracts, while preserving `xiuxian-vector` API
    names through type aliases and validating fusion/search regression lanes
    (`43 passed`).
374. Use `387-knowledge-category-sink-to-xiuxian-types-and-cross-crate-alignment-2026-03-05.md`
    for sinking duplicated `KnowledgeCategory` enums from `xiuxian-skills`
    and `xiuxian-wendao` into `xiuxian-types`, preserving scanner/storage
    behavior through explicit defaults and plural-key mapping helpers, and
    validating strict clippy plus targeted knowledge regression lanes
    (`9 + 20 passed`).
375. Use `388-workspace-schemars-12-unification-and-zhenfa-macro-compat-2026-03-05.md`
    for unifying workspace crates onto `schemars 1.2.0`, removing per-crate
    version drift, and fixing `zhenfa_tool` macro compatibility (`schema_for!`
    migration from `schema.schema` to direct schema serialization), validated
    with strict clippy plus schema/macro/router regression lanes
    (`3 + 3 + 7 passed`).
376. Use `389-omni-mcp-client-physical-sink-into-xiuxian-mcp-2026-03-05.md`
    for completing MCP package-boundary convergence by physically removing the
    legacy `omni-mcp-client` crate and retaining a single authoritative MCP
    client surface in `xiuxian-mcp`, validated via residual-audit, check,
    clippy, and MCP test lanes (`17 passed`, `1 skipped`).

377. Use `409-xiuxian-wendao-hybrid-retriever-rust-phase1-2026-03-07.md`
    for the first pure-Rust hybrid-retriever slice in `xiuxian-wendao`,
    landing typed quantum-fusion orchestration, `PageIndex` ancestry
    traceability, bounded PPR score fusion, and targeted Rust-only
    validation evidence (`cargo check`, `clippy`, and `nextest`).

378. Use `410-xiuxian-wendao-semantic-ignition-rust-seam-phase2-2026-03-07.md`
    for the second pure-Rust hybrid-retriever slice in `xiuxian-wendao`,
    adding a backend-agnostic semantic ignition seam, request normalization,
    empty-signal short-circuiting, and targeted seam/regression validation
    evidence (`3 + 2 passed`).

379. Use `411-xiuxian-wendao-vector-adapter-and-seam-boundary-convergence-2026-03-07.md`
    for the third pure-Rust hybrid-retriever slice, covering the new
    `xiuxian-wendao-vector` integration crate, removal of over-strong
    `Send/Sync` seam bounds, metadata-preserving vector projection, and
    validated adapter plus Wendao regression lanes (`5 + 3 + 2 passed`).

380. Use `412-xiuxian-wendao-semantic-policy-filters-and-score-gates-2026-03-07.md`
    for the fourth pure-Rust hybrid-retriever slice, adding typed adapter
    metadata filters, explicit summary-only semantic scope, request-level
    minimum semantic score gating, and clean validation evidence
    (`8 + 4 passed`).

381. Use `413-xiuxian-wendao-retrieval-plan-semantic-policy-inputs-2026-03-07.md`
    for the fifth pure-Rust hybrid-retriever slice, adding first-class
    semantic-policy inputs to Wendao planning, retrieval-plan schema exposure,
    runtime default merging, and targeted validation evidence (`3 + 1 + 1 passed`).

382. Use `414-xiuxian-wendao-vector-retrieval-plan-execution-bridge-2026-03-07.md`
    for the sixth pure-Rust hybrid-retriever slice, closing the gap between
    Wendao retrieval plans and vector-execution setup with graph-only
    short-circuiting, canonical plan consumption, and validated adapter gates
    (`13 passed`).

383. Use `415-xiuxian-wendao-vector-hybrid-search-runtime-entry-point-2026-03-07.md`
    for the seventh pure-Rust hybrid-retriever slice, adding a real hybrid-search
    runtime entry point in `xiuxian-wendao-vector`, retrieval-plan-governed
    semantic gating, budget-aligned semantic request sizing, and validated
    runtime lanes (`15 passed`).

384. Use `416-xiuxian-wendao-vector-deferred-query-vectorization-seam-2026-03-07.md`
    for the eighth pure-Rust hybrid-retriever slice, adding a text-first hybrid
    search seam, deferred query vectorization after planning, normalized planned
    query vectorization, and validated runtime/error lanes (`18 passed`).

385. Use `417-xiuxian-daochang-zhenfa-hybrid-query-vectorizer-adapter-2026-03-07.md`
    for the ninth pure-Rust hybrid-retriever slice, adding the first real
    `xiuxian-daochang` query-vectorizer adapter over `EmbeddingClient`, black-box
    adapter validation, and crate-level convergence evidence (`19 passed, 1 skipped`).

386. Use `418-xiuxian-daochang-hybrid-wendao-search-runtime-and-typed-semantic-export-2026-03-07.md`
    for the tenth pure-Rust hybrid-retriever slice, exporting typed
    `PageIndex`-derived semantic documents from `xiuxian-wendao`, wiring a real
    `xiuxian-daochang` `wendao.search` hybrid runtime over `EmbeddingClient`,
    and validating both the Wendao export lane and the caller runtime lane
    (`1 passed, 72 skipped`; `20 passed, 1 skipped`).

387. Use `419-xiuxian-vector-send-safe-keyword-index-and-hybrid-future-boundaries-2026-03-07.md`
    for the eleventh pure-Rust hybrid-retriever slice, replacing
    `Rc`/`RefCell` in shared vector runtime state with `Arc`/`Mutex`, restoring
    `Send` on the hybrid seam future aliases, and adding compile-time
    thread-safety regression guards (`2 passed`; `4 passed, 69 skipped`; `2 passed`; `3 passed`).

388. Use `420-xiuxian-wendao-arrow-native-batch-quantum-scorer-2026-03-07.md`
    for the twelfth pure-Rust hybrid-retriever slice, adding an Arrow-native
    `BatchQuantumScorer` in `xiuxian-wendao`, typed batch-fusion errors,
    schema-metadata preservation, and focused columnar scoring regression
    evidence (`3 passed, 73 skipped`; `2 passed, 74 skipped`).

389. Use `421-xiuxian-wendao-quantum-fusion-orchestration-batch-scorer-integration-2026-03-07.md`
    for the thirteenth pure-Rust hybrid-retriever slice, wiring
    `quantum_contexts_from_anchors(...)` onto the Arrow-native batch scorer,
    separating candidate preparation from score fusion, and validating the real
    orchestration path with duplicate-anchor regression coverage
    (`3 passed, 74 skipped`; `3 passed, 74 skipped`).

390. Use `422-xiuxian-wendao-typed-quantum-orchestration-errors-and-fallback-removal-2026-03-07.md`
    for the fourteenth pure-Rust hybrid-retriever slice, removing the last
    scalar fallback from `quantum_fusion`, introducing typed orchestration and
    semantic-ignition error layers, and validating the new boundary across
    Wendao, Wendao-Vector, and Daochang (`3 passed, 74 skipped`; `4 passed, 73
    skipped`; `4 passed`; `2 passed`; `20 passed, 1 skipped`).

391. Use `423-xiuxian-wendao-batch-native-quantum-anchor-batch-entry-2026-03-07.md`
    for the fifteenth pure-Rust hybrid-retriever slice, adding a first-class
    batch-native orchestration entry for prepared Arrow anchor batches,
    preserving raw row identity during saliency fusion, and validating custom
    input-column and wrong-type regressions (`2 passed, 77 skipped`; `3 passed,
    76 skipped`; `4 passed, 75 skipped`).

392. Use `424-xiuxian-wendao-shared-quantum-anchor-batch-contract-2026-03-07.md`
    for the sixteenth pure-Rust hybrid-retriever slice, centralizing the Arrow
    anchor-batch contract in a shared validated view, removing duplicated batch
    validation across orchestration and scoring, and expanding input-contract
    regression coverage (`3 passed, 79 skipped`; `5 passed, 77 skipped`; `3
    passed, 79 skipped`; `4 passed, 78 skipped`).

393. Use `425-xiuxian-wendao-semantic-anchor-resolution-stage-2026-03-07.md`
    for the seventeenth pure-Rust hybrid-retriever slice, extracting semantic
    anchor resolution into its own stage module, simplifying orchestration into
    clearer pipeline composition, and validating whitespace-padded doc fallbacks
    through the batch-native path (`3 passed, 80 skipped`; `6 passed, 77
    skipped`; `3 passed, 80 skipped`; `4 passed, 79 skipped`).

394. Use `426-xiuxian-wendao-scored-context-reconstruction-stage-2026-03-07.md`
    for the eighteenth pure-Rust hybrid-retriever slice, extracting scored-batch
    decoding and final `QuantumContext` reconstruction into its own stage,
    keeping `orchestrate.rs` focused on pipeline coordination, and validating
    duplicate-row batch-native reconstruction (`3 passed, 81 skipped`; `7
    passed, 77 skipped`; `3 passed, 81 skipped`; `4 passed, 80 skipped`).

395. Use `427-xiuxian-wendao-topology-expansion-stage-2026-03-07.md`
    for the nineteenth pure-Rust hybrid-retriever slice, extracting graph
    expansion and topology aggregation into a dedicated stage, further reducing
    `orchestrate.rs` to stage composition, and strengthening batch-native
    related-cluster regression coverage (`3 passed, 81 skipped`; `7 passed, 77
    skipped`; `3 passed, 81 skipped`; `4 passed, 80 skipped`).

396. Use `428-xiuxian-wendao-link-graph-fixture-expected-snapshot-migration-2026-03-07.md`
    for the LinkGraph test-architecture slice that moves hybrid retrieval tests
    to `tests/fixtures/<suite>/input` plus `tests/fixtures/<suite>/expected`,
    adds shared hybrid corpus materialization and JSON projection support, and
    leaves the migrated `test_link_graph` lane strict-clippy clean (`13 passed,
    71 skipped`; `3 passed, 81 skipped`; `1 passed, 83 skipped`).

397. Use `429-xiuxian-wendao-semantic-ignition-fixture-expected-contracts-2026-03-07.md`
    for the follow-up LinkGraph test-architecture slice that migrates
    `semantic_ignition` onto the shared hybrid fixture and expected-contract
    model, covering call-count, empty-request, min-score, and backend-error
    behavior (`4 passed, 80 skipped`; strict `test_link_graph` clippy clean).

398. Use `430-xiuxian-wendao-page-index-fixture-expected-contracts-2026-03-07.md`
    for the page-index test-architecture slice that adds shared `LinkGraph`
    fixture-tree materialization, migrates the full `page_index` suite onto
    per-scenario input/expected contracts, and keeps `test_link_graph` strict
    clippy clean (`6 passed, 78 skipped`).

399. Use `431-xiuxian-wendao-build-scope-fixture-expected-contracts-2026-03-07.md`
    for the build-scope test-architecture slice that migrates directory-filter
    and skill-metadata promotion coverage onto per-scenario input/expected
    contracts while keeping the `test_link_graph` lane strict-clippy clean (`4
    passed, 80 skipped`).

400. Use `432-xiuxian-wendao-search-core-fixture-expected-contracts-2026-03-07.md`
    for the search-core test-architecture slice that migrates the central
    `LinkGraph` search lane onto per-scenario `input/expected` fixtures,
    captures both hit lists and planned-payload contracts as JSON, and keeps
    the `test_link_graph` lane strict-clippy clean (`9 passed, 75 skipped`).

401. Use `433-xiuxian-wendao-search-match-strategies-fixture-expected-contracts-2026-03-07.md`
    for the search-match-strategies test-architecture slice that migrates
    path-fuzzy, exact, and regex coverage onto per-scenario `input/expected`
    fixtures, keeps projections focused on stable semantic fields, and leaves
    the `test_link_graph` lane strict-clippy clean (`6 passed, 78 skipped`).

402. Use `434-xiuxian-wendao-graph-navigation-fixture-expected-contracts-2026-03-07.md`
    for the graph-navigation test-architecture slice that migrates traversal,
    metadata, TOC, and related-diagnostics coverage onto per-scenario
    `input/expected` fixtures while normalizing timing-like metrics into stable
    semantic invariants (`3 passed, 81 skipped`).

403. Use `435-xiuxian-wendao-markdown-attachments-fixture-expected-contracts-2026-03-07.md`
    for the markdown-attachments test-architecture slice that migrates link and
    attachment parsing coverage onto per-scenario `input/expected` fixtures,
    keeps attachment-hit contracts score-agnostic, and leaves the
    `test_link_graph` lane strict-clippy clean (`5 passed, 79 skipped`).

404. Use `436-xiuxian-wendao-refresh-fixture-expected-contracts-2026-03-07.md`
    for the refresh test-architecture slice that migrates incremental update,
    deletion, and threshold-escalation coverage onto per-scenario
    `input/expected` fixtures and captures refresh as a serialized
    state-transition contract (`3 passed, 81 skipped`).

405. Use `437-xiuxian-wendao-semantic-policy-fixture-expected-contracts-2026-03-07.md`
    for the semantic-policy test-architecture slice that migrates directive and
    planned-payload policy propagation checks onto expected JSON contracts
    (`3 passed, 81 skipped`).

406. Use `438-xiuxian-wendao-search-filters-fixture-expected-contracts-2026-03-07.md`
    for the search-filters test-architecture slice that migrates the remaining
    inline filter scenarios onto per-scenario `input/expected` fixtures and
    keeps the contracts path-centric instead of score-centric (`7 passed, 77
    skipped`).

407. Use `439-xiuxian-wendao-tree-scope-fixture-expected-contracts-2026-03-07.md`
    for the tree-scope test-architecture slice that migrates section-scope and
    edge-type filter scenarios onto per-scenario `input/expected` fixtures with
    stable section-label and per-path-count contracts (`9 passed, 75 skipped`).

408. Use `440-xiuxian-wendao-cache-build-fixture-expected-contracts-2026-03-07.md`
    for the cache-build test-architecture slice that migrates snapshot reuse,
    cache invalidation, and saliency seeding coverage onto fixture-backed
    expected contracts and leaves only mutation-oriented writes in the
    `test_link_graph` lane (`3 passed, 81 skipped`).

409. Use `441-xiuxian-wendao-skill-vfs-fixture-contracts-2026-03-07.md`
    for the Skill-VFS test-architecture slice that migrates resolver and
    internal-manifest coverage onto scenario-based `tests/fixtures/skill_vfs`
    contracts, splits materialization from write-based seeding helpers, and
    keeps the targeted lanes strict-clippy clean (`17 passed, 0 skipped`).

410. Use `442-xiuxian-wendao-skill-vfs-snapshot-to-fixture-contracts-2026-03-07.md`
    for the Skill-VFS contract lane that replaces the dedicated
    `tests/snapshots/skill_vfs` root with fixture-backed `input/expected`
    scenarios, extracts domain-specific contract projection helpers, and keeps
    the renamed `test_skill_vfs_contracts` lane strict-clippy clean (`7 passed,
    0 skipped`).

411. Use `443-xiuxian-wendao-internal-skill-authority-fixture-contracts-2026-03-07.md`
    for the internal-skill authority lane that migrates authorization, catalog
    fast-path, alias-preparation, validation-failure, and empty-root coverage
    onto fixture-backed `tests/fixtures/skill_vfs` contracts and keeps the lane
    strict-clippy clean (`6 passed, 0 skipped`).

412. Use `444-xiuxian-wendao-small-snapshot-lanes-to-fixture-contracts-2026-03-07.md`
    for the small Wendao snapshot lanes that rename URI, skill-semantics, and
    embedded-resource-registry tests to `*_contracts.rs`, relocate expected
    outputs under `tests/fixtures/.../expected`, and remove the obsolete
    snapshot files (`3 passed, 0 skipped`).

413. Use `445-xiuxian-wendao-asset-and-registry-snapshot-lanes-to-fixture-contracts-2026-03-07.md`
    for the Wendao asset-request, embedded-skill-api, and dynamic-discovery
    lanes that rename `*_snapshots.rs` tests to `*_contracts.rs`, relocate
    expected outputs into fixture trees, and remove the obsolete snapshot files
    (`3 passed, 0 skipped`).

414. Use `446-xiuxian-wendao-repository-internal-manifest-contract-fixture-2026-03-07.md`
    for the repository-backed internal-manifest regression lane that renames the
    test to `*_contracts.rs`, relocates its expected state under
    `tests/fixtures/skill_vfs/.../expected`, and removes the last
    repository-internal-manifest snapshot file (`1 passed, 0 skipped`).

415. Use `447-xiuxian-wendao-parser-contract-fixture-migration-2026-03-07.md`
    for the parser lane that renames the old snapshot suite to
    `test_parser_contracts.rs`, moves markdown and ORG placeholder expectations
    under `tests/fixtures/parser/...`, and removes the final
    `snapshot_assertions` dependency from `xiuxian-wendao/tests` (`6 passed, 1
    skipped`).

416. Use `448-xiuxian-wendao-link-graph-inline-corpus-fixture-contracts-2026-03-07.md`
    for the remaining small link-graph inline-corpus lanes that move
    weighted-seed PPR, mixed-topology traversal, and bounded
    agentic-expansion telemetry onto fixture-backed `input/expected`
    contracts and keep the targeted suite strict-clippy clean (`5 passed,
    0 skipped`).

417. Use `449-xiuxian-wendao-ppr-and-cli-related-fixture-contracts-2026-03-07.md`
    for the weighted-seed PPR precision lane and the `wendao related` CLI
    surface that now reuse fixture-backed `input/expected` contracts, remove
    imperative related-diagnostics helpers, and keep the targeted suite
    strict-clippy clean (`37 passed, 0 skipped`).

418. Use `450-xiuxian-wendao-seed-and-priors-fixture-contracts-2026-03-07.md`
    for the `test_link_graph_seed_and_priors` lane that moves seed-cluster
    grounding, structural-prior boosting, and journal-driven agenda retrieval
    onto fixture-backed contracts while restoring `mod.rs` to interface-only
    responsibility (`3 passed, 0 skipped`).

419. Use `451-xiuxian-wendao-cli-search-basic-fixture-contracts-2026-03-07.md`
    for the `test_wendao_cli/search/basic` lane that moves the default,
    path-fuzzy, sorted, and verbose search surfaces onto fixture-backed CLI
    contracts with a lane-specific payload projection module (`4 passed, 32
    skipped`).

420. Use `452-xiuxian-wendao-cli-search-directives-fixture-contracts-2026-03-07.md`
    for the `test_wendao_cli/search/directives` lane that moves directive,
    filter, temporal, and legacy-error search surfaces onto fixture-backed CLI
    contracts with stable error semantics (`5 passed, 31 skipped`).

421. Use `453-xiuxian-wendao-cli-search-link-filters-and-provisional-fixture-contracts-2026-03-07.md`
    for the remaining `test_wendao_cli/search` lanes that move link-filter and
    provisional-overlay behavior onto fixture-backed CLI contracts with
    lane-specific support modules (`4 passed, 32 skipped`).

422. Use `454-xiuxian-wendao-refresh-mode-fixture-deduplication-2026-03-07.md`
    for the link-graph refresh mode consolidation that removes the redundant
    legacy test binary, expands the fixture-backed threshold contract to cover
    `noop`, `full`, and `delta`, and preserves their graph-state differences
    (`2 passed, 82 skipped`).

423. Use `455-xiuxian-wendao-skill-vfs-resolver-contract-deduplication-2026-03-07.md`
    for the Skill VFS resolver consolidation that removes the duplicate
    `test_skill_vfs_resolver` integration file, folds missing-resource semantics
    into the existing fixture-backed resolver contract, and deletes dead fixture
    scenarios (`7 passed, 0 skipped`).

424. Use `456-xiuxian-wendao-skill-vfs-uri-and-asset-contract-deduplication-2026-03-07.md`
    for the Skill VFS URI and asset-request consolidation that removes the
    duplicate `test_skill_vfs_uri` and `test_asset_request_api` integration
    files while preserving plain stripped-body behavior inside the contract
    surface (`2 passed, 0 skipped`).

425. Use `457-xiuxian-wendao-sync-suite-deduplication-and-mod-interface-restoration-2026-03-07.md`
    for the sync-suite consolidation that removes `sync_unit`, migrates the
    remaining policy and glob-pattern tests into `test_sync`, and restores
    `test_sync/mod.rs` to interface-only responsibility (`13 passed, 0 skipped`).

426. Use `458-xiuxian-wendao-skill-reference-semantics-contract-deduplication-2026-03-07.md`
    for the skill-reference semantics consolidation that removes the duplicate
    assertion-only test binary in favor of the existing fixture-backed
    classification matrix contract (`1 passed, 0 skipped`).

427. Use `459-xiuxian-wendao-registry-contract-deduplication-2026-03-07.md`
    for the embedded Wendao registry consolidation that removes the duplicate
    dynamic-discovery and resource-registry top-level test binaries in favor of
    the existing fixture-backed contract surfaces (`2 passed, 0 skipped`).

428. Use `460-xiuxian-wendao-link-graph-saliency-support-module-boundary-2026-03-07.md`
    for the `test_link_graph_saliency` module cleanup that extracts shared
    Valkey helpers into `support.rs`, restores `mod.rs` to interface-only
    responsibility, and localizes imports inside each child test (`5 passed, 0 skipped`).

429. Use `461-xiuxian-wendao-link-graph-agentic-support-module-boundary-2026-03-07.md`
    for the `test_link_graph_agentic` module cleanup that extracts shared
    Valkey helpers into `support.rs`, restores `mod.rs` to interface-only
    responsibility, and localizes imports inside each child test (`5 passed, 0 skipped`).

430. Use `462-xiuxian-wendao-knowledge-mod-import-bucket-removal-2026-03-07.md`
    for the `test_knowledge` cleanup that removes the `mod.rs` import bucket
    and replaces `use super::*;` with explicit per-file domain imports (`13 passed, 0 skipped`).

431. Use `463-xiuxian-wendao-benchmark-and-parser-support-module-boundaries-2026-03-07.md`
    for the benchmark and parser cleanup that extracts shared generators,
    budgets, and fixture-path helpers into `support.rs` files and restores each
    `mod.rs` to interface-only responsibility (`19 passed, 1 skipped`).

432. Use `464-xiuxian-wendao-graph-support-module-boundary-2026-03-07.md`
    for the `test_graph` cleanup that extracts the Valkey gate into `support.rs`,
    restores `mod.rs` to interface-only responsibility, and localizes graph
    type imports inside each child test (`25 passed, 0 skipped`).

433. Use `465-xiuxian-wendao-cli-test-support-module-boundaries-2026-03-07.md`
    for the `test_wendao_cli` cleanup that restores CLI module roots to
    interface-only responsibility, extracts lane-specific support modules, and
    localizes child-test imports (`36 passed, 0 skipped`).

434. Use `466-xiuxian-wendao-link-graph-support-boundaries-and-page-index-validation-2026-03-07.md`
    for the remaining `test_link_graph` and link-graph benchmark cleanup that
    adds focused support modules, removes ambient parent imports, records a
    zero-warning `cargo check`, and captures the current four-test page-index
    snapshot validation exception (`80 passed, 4 failed`, benchmarks `0 passed, 2 skipped`).

435. Use `467-xiuxian-wendao-page-index-line-range-contract-realignment-2026-03-08.md`
    for the page-index fixture follow-up that realigns `line_range` snapshots to
    the current 1-based inclusive physical-span contract and closes the last
    four `test_link_graph` failures (`84 passed, 0 skipped`; combined `120 passed, 2 skipped`).

## Maintenance Rules

- Keep this directory English-only.
- Keep tracking feature-based and evidence-driven.
- Update snapshot and scorecard whenever quality signals change.


436. Use `468-xiuxian-daochang-telegram-runtime-test-support-modularization-2026-03-08.md`
    for the `xiuxian-daochang` Telegram runtime test cleanup that restores the
    suite root to interface-only responsibility, decomposes the shared harness
    into focused `support/` submodules, records the passing targeted nextest
    run (`77 passed, 0 skipped`), and isolates the remaining follow-up blocker
    to unrelated OCR compile debt elsewhere in the crate.

437. Use `469-xiuxian-memory-engine-common-test-support-modularization-2026-03-08.md`
    for the `xiuxian-memory-engine` test-support cleanup that restores
    `tests/common/mod.rs` to interface-only responsibility, moves the path
    helper into `tests/common/paths.rs`, and records a clean validation sweep
    (`69 passed, 0 skipped`).

438. Use `470-xiuxian-lance-integration-test-entrypoint-rename-2026-03-08.md`
    for the `xiuxian-lance` cleanup that replaces the generic `tests/mod.rs`
    integration entrypoint with an explicit `test_vector_record_batch_reader.rs`
    binary and records a clean validation sweep (`7 passed, 0 skipped`).

439. Use `471-xiuxian-memory-engine-test-memory-engine-modularization-2026-03-08.md`
    for the `xiuxian-memory-engine` large-suite cleanup that decomposes the
    700-line `test_memory_engine.rs` entrypoint into focused directory modules
    and records a clean targeted validation sweep (`16 passed, 0 skipped`).

440. Use `472-xiuxian-memory-engine-test-complex-scenarios-modularization-2026-03-08.md`
    for the `xiuxian-memory-engine` scenario-suite cleanup that decomposes the
    large `test_complex_scenarios.rs` entrypoint into focused directory modules
    and records a clean targeted validation sweep (`10 passed, 0 skipped`).

441. Use `473-xiuxian-wendao-test-enhancer-modularization-2026-03-08.md`
    for the `xiuxian-wendao` enhancer-suite cleanup that decomposes the mixed
    `test_enhancer.rs` entrypoint into focused directory modules and records a
    clean targeted validation sweep (`15 passed, 0 skipped`).

442. Use `474-xiuxian-wendao-ingress-spider-unit-modularization-2026-03-08.md`
    for the `xiuxian-wendao` spider-ingress cleanup that separates URI
    contracts from bridge-ingestion behavior, extracts test-only recording
    fixtures into `support.rs`, and records a clean targeted validation sweep
    (`6 passed, 0 skipped`).

443. Use `475-xiuxian-wendao-test-zhenfa-native-tools-modularization-2026-03-08.md`
    for the feature-gated `xiuxian-wendao` zhenfa-router cleanup that splits
    native dispatch, hit-type classification, cache-key, and request-context
    coverage into focused directory modules and records a clean feature-aware
    validation sweep (`8 passed, 0 skipped`).

444. Use `476-xiuxian-wendao-page-index-parent-anchor-map-2026-03-08.md`
    for the `xiuxian-wendao` page-index anchor enhancement that adds
    `PageIndexNode.parent_id`, introduces `LinkGraphIndex.node_parent_map`,
    keeps cache snapshots topology-compatible, and records a clean targeted
    `test_link_graph` validation sweep (`85 passed, 0 skipped`).

445. Use `477-xiuxian-wendao-skill-vfs-contract-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` Skill VFS contract-suite cleanup that decomposes
    the mixed `test_skill_vfs_contracts.rs` entrypoint into focused directory
    modules and records a clean targeted validation sweep (`7 passed, 0 skipped`).

446. Use `478-xiuxian-wendao-internal-skill-authority-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` internal skill authority cleanup that decomposes
    the mixed `test_internal_skill_authority.rs` entrypoint into focused
    directory modules and records a clean targeted validation sweep (`6 passed, 0 skipped`).

447. Use `479-xiuxian-wendao-link-graph-refs-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` link-graph reference cleanup that decomposes the
    mixed `test_link_graph_refs.rs` entrypoint into focused directory modules
    and records a clean targeted validation sweep (`15 passed, 0 skipped`).

448. Use `480-xiuxian-wendao-intent-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` intent extractor cleanup that decomposes the mixed
    `test_intent.rs` entrypoint into focused directory modules and records a
    clean targeted validation sweep (`14 passed, 0 skipped`).

449. Use `481-xiuxian-wendao-internal-skill-manifest-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` internal skill manifest cleanup that decomposes
    the mixed `test_internal_skill_manifest.rs` entrypoint into focused
    directory modules and records a clean targeted validation sweep (`4 passed, 0 skipped`).

450. Use `482-xiuxian-wendao-markdown-syntax-algorithm-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` markdown syntax fixture cleanup that decomposes
    the mixed `test_markdown_syntax_algorithm_fixtures.rs` entrypoint into
    focused directory modules and records a clean targeted validation sweep (`5 passed, 0 skipped`).

451. Use `483-xiuxian-wendao-kg-cache-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` KG cache cleanup that decomposes the mixed
    `test_kg_cache.rs` entrypoint into focused directory modules and records a
    clean targeted validation sweep (`4 passed, 0 skipped`).

452. Use `484-xiuxian-wendao-storage-unit-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` storage integration cleanup that decomposes the
    mixed `storage_unit.rs` entrypoint into focused directory modules and
    records a clean targeted validation sweep (`4 passed, 0 skipped`).

453. Use `485-xiuxian-wendao-entity-unit-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` entity model cleanup that decomposes the mixed
    `entity_unit.rs` entrypoint into focused directory modules, corrects a stale
    relation-id assertion, and records a clean targeted validation sweep (`5 passed, 0 skipped`).

454. Use `486-xiuxian-wendao-types-unit-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` types cleanup that decomposes the mixed
    `types_unit.rs` entrypoint into focused directory modules and records a
    clean targeted validation sweep (`3 passed, 0 skipped`).

455. Use `487-xiuxian-wendao-dependency-indexer-symbols-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` dependency indexer symbol cleanup that decomposes
    the mixed `dependency_indexer_symbols_unit.rs` entrypoint into focused
    directory modules and records a clean targeted validation sweep (`4 passed, 0 skipped`).

456. Use `488-xiuxian-wendao-dependency-debug-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` dependency debug cleanup that decomposes the mixed
    `test_dependency_debug.rs` entrypoint into focused directory modules and
    records a clean targeted validation sweep (`4 passed, 0 skipped`).

457. Use `489-xiuxian-wendao-hmas-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` HMAS validation cleanup that decomposes the mixed
    `test_hmas.rs` entrypoint into focused directory modules and records a
    clean targeted validation sweep (`4 passed, 0 skipped`).

458. Use `490-xiuxian-wendao-dependency-indexer-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` dependency indexer cleanup that decomposes the
    mixed `test_dependency_indexer.rs` entrypoint into focused directory
    modules and records a clean targeted validation sweep (`4 passed, 0 skipped`).

459. Use `491-xiuxian-wendao-dependency-indexer-indexer-unit-modularization-2026-03-08.md`
    for the `xiuxian-wendao` dependency indexer core cleanup that decomposes
    the mixed `dependency_indexer_indexer_unit.rs` entrypoint into focused
    directory modules and records a clean targeted validation sweep (`3 passed, 0 skipped`).

460. Use `492-xiuxian-wendao-dep-indexer-py-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` dep-indexer Python support cleanup that decomposes
    the mixed `dep_indexer_py.rs` entrypoint into focused directory modules and
    records a clean targeted validation sweep (`4 passed, 0 skipped`).

461. Use `493-xiuxian-wendao-unified-symbol-unit-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` unified-symbol cleanup that decomposes the mixed
    `unified_symbol_unit.rs` entrypoint into focused directory modules and
    records a clean targeted validation sweep (`4 passed, 0 skipped`).

462. Use `494-xiuxian-wendao-zhenfa-router-xml-lite-test-modularization-2026-03-08.md`
    for the feature-gated `xiuxian-wendao` XML-lite hit-type cleanup that
    decomposes the mixed `zhenfa_router_xml_lite_unit.rs` entrypoint into
    focused directory modules and records a clean feature-aware validation sweep
    (`5 passed, 0 skipped`).

463. Use `495-xiuxian-wendao-zhenfa-router-integration-test-modularization-2026-03-08.md`
    for the feature-gated `xiuxian-wendao` router integration cleanup that
    decomposes the mixed `test_zhenfa_router.rs` entrypoint into focused
    directory modules and records a clean feature-aware validation sweep (`3 passed, 0 skipped`).

464. Use `496-xiuxian-wendao-link-graph-agentic-expansion-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` link-graph agentic expansion cleanup that
    decomposes the mixed `test_link_graph_agentic_expansion.rs` entrypoint into
    focused directory modules and records a clean targeted validation sweep (`3 passed, 0 skipped`).

465. Use `497-xiuxian-wendao-unified-symbol-py-test-modularization-2026-03-08.md`
    for the `xiuxian-wendao` unified-symbol Python-support cleanup that
    decomposes the mixed `unified_symbol_py.rs` entrypoint into focused
    directory modules and records a clean targeted validation sweep (`3 passed, 0 skipped`).

466. Use `498-xiuxian-ast-extract-test-modularization-2026-03-08.md`
    for the `xiuxian-ast` extraction-suite cleanup that decomposes the mixed
    `test_extract.rs` entrypoint into focused directory modules and records a
    clean crate-wide validation sweep (`74 passed, 0 skipped`).

467. Use `499-xiuxian-ast-python-tree-sitter-test-modularization-2026-03-08.md`
    for the `xiuxian-ast` tree-sitter parser cleanup that decomposes the mixed
    `test_python_tree_sitter.rs` entrypoint into focused directory modules and
    records a clean crate-wide validation sweep (`74 passed, 0 skipped`).

468. Use `500-xiuxian-ast-benchmark-test-modularization-2026-03-08.md`
    for the `xiuxian-ast` benchmark-suite cleanup that decomposes the mixed
    `test_ast_benchmark.rs` entrypoint into focused directory modules and
    records a clean crate-wide validation sweep (`74 passed, 0 skipped`).

469. Use `501-xiuxian-ast-generic-tests-mod-entrypoint-removal-2026-03-08.md`
    for the `xiuxian-ast` cleanup that removes the redundant generic
    `tests/mod.rs` entrypoint and records a clean crate-wide validation sweep
    (`74 passed, 0 skipped`).

470. Use `502-xiuxian-ast-lang-test-modularization-2026-03-08.md`
    for the `xiuxian-ast` language-helper cleanup that decomposes the compact
    but mixed `test_lang.rs` entrypoint into focused directory modules and
    records a clean targeted validation sweep (`3 passed, 0 skipped`).

471. Use `503-xiuxian-config-core-paths-test-modularization-2026-03-08.md`
    for the `xiuxian-config-core` path-helper cleanup that decomposes the
    compact but mixed `paths_unit.rs` entrypoint into focused directory modules
    and records a clean crate-wide validation sweep (`12 passed, 0 skipped`).

472. Use `504-xiuxian-config-core-resolve-test-modularization-2026-03-08.md`
    for the `xiuxian-config-core` resolver cleanup that decomposes the mixed
    `test_resolve.rs` entrypoint into focused directory modules and records a
    clean crate-wide validation sweep (`12 passed, 0 skipped`).

473. Use `505-xiuxian-config-core-cache-test-modularization-2026-03-08.md`
    for the `xiuxian-config-core` cache cleanup that decomposes `test_cache.rs`
    into focused directory modules, makes the shared spec lifetime explicit,
    and records a clean crate-wide validation sweep (`12 passed, 0 skipped`).

474. Use `506-xiuxian-tokenizer-core-test-modularization-2026-03-08.md`
    for the `xiuxian-tokenizer` core-suite cleanup that decomposes the mixed
    `test_tokenizer.rs` entrypoint into focused directory modules and records a
    clean crate-wide validation sweep (`16 passed, 0 skipped`).

475. Use `507-xiuxian-tokenizer-benchmark-test-modularization-2026-03-08.md`
    for the `xiuxian-tokenizer` benchmark-suite cleanup that decomposes the
    mixed `test_tokenizer_benchmark.rs` entrypoint into focused directory
    modules and records a clean crate-wide validation sweep (`16 passed, 0 skipped`).

476. Use `508-xiuxian-tokenizer-generic-tests-mod-entrypoint-removal-2026-03-08.md`
    for the `xiuxian-tokenizer` cleanup that removes the redundant generic
    `tests/mod.rs` entrypoint and records a clean crate-wide validation sweep
    (`16 passed, 0 skipped`).

477. Use `509-xiuxian-zhenfa-error-mapping-test-modularization-2026-03-08.md`
    for the `xiuxian-zhenfa` error-surface cleanup that decomposes the compact
    but mixed `test_error_mapping.rs` entrypoint into focused directory modules
    and records a clean crate-wide validation sweep (`32 passed, 0 skipped`).

478. Use `510-xiuxian-zhenfa-context-extensions-test-modularization-2026-03-08.md`
    for the `xiuxian-zhenfa` context-extension cleanup that decomposes
    `test_context_extensions.rs` into focused directory modules and records a
    clean crate-wide validation sweep (`32 passed, 0 skipped`).

479. Use `511-xiuxian-zhenfa-contracts-test-modularization-2026-03-08.md`
    for the `xiuxian-zhenfa` JSON-RPC contract cleanup that decomposes
    `test_contracts.rs` into focused directory modules and records a clean
    crate-wide validation sweep (`32 passed, 0 skipped`).

480. Use `512-xiuxian-zhenfa-xml-lite-test-modularization-2026-03-08.md`
    for the `xiuxian-zhenfa` XML-lite helper cleanup that converts
    `test_xml_lite.rs` into a thin launcher backed by a directory module and
    records a clean crate-wide validation sweep (`32 passed, 0 skipped`).

481. Use `513-xiuxian-zhenfa-transmuter-test-modularization-2026-03-08.md`
    for the `xiuxian-zhenfa` transmuter cleanup that decomposes
    `test_transmuter.rs` into focused directory modules and records a clean
    crate-wide validation sweep (`32 passed, 0 skipped`).

482. Use `514-xiuxian-zhenfa-zhenfa-tool-macro-test-modularization-2026-03-08.md`
    for the `xiuxian-zhenfa` macro-suite cleanup that decomposes
    `test_zhenfa_tool_macro.rs` into focused directory modules and records a
    clean crate-wide validation sweep (`32 passed, 0 skipped`).

483. Use `515-xiuxian-zhenfa-native-registry-test-modularization-2026-03-08.md`
    for the `xiuxian-zhenfa` native-registry cleanup that decomposes the large
    mixed `test_native_registry.rs` entrypoint into focused directory modules
    and records a clean crate-wide validation sweep (`32 passed, 0 skipped`).

484. Use `516-xiuxian-wendao-hierarchical-uplink-activation-2026-03-08.md`
    for the `xiuxian-wendao` hierarchical-uplink activation that introduces
    `HierarchicalHit`, centralizes lineage extraction under
    `link_graph/index/search/hierarchical.rs`, routes quantum anchor resolution
    through that contract, and records a clean focused validation sweep
    (`87 passed, 0 skipped`).

485. Use `517-xiuxian-wendao-quantum-hierarchical-contract-consolidation-2026-03-08.md`
    for the `xiuxian-wendao` quantum-fusion contract cleanup that threads
    `HierarchicalHit` through resolved anchors and scored candidates, removes
    duplicated lineage fields from the internal pipeline, and records a clean
    focused validation sweep (`87 passed, 0 skipped`).

486. Use `518-xiuxian-wendao-quantum-context-traceability-fields-2026-03-08.md`
    for the `xiuxian-wendao` hybrid-result contract expansion that adds stable
    `doc_id` and `path` fields to `QuantumContext`, snapshots those fields
    across hybrid fixtures, and records a clean focused validation sweep
    (`87 passed, 0 skipped`).

487. Use `519-xiuxian-daochang-hybrid-traceability-rendering-2026-03-08.md`
    for the `xiuxian-daochang` downstream hybrid-rendering cleanup that
    surfaces `QuantumContext.doc_id/path` in semantic XML-lite hits, locks the
    consumer contract with integration assertions, and records a clean focused
    validation sweep (`20 passed, 1 skipped`).
