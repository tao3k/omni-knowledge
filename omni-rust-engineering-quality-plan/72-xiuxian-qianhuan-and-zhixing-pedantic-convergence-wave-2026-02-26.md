# Xiuxian-Qianhuan And Zhixing Pedantic Convergence Wave (2026-02-26)

## Scope

Converge `xiuxian-qianhuan` and `xiuxian-zhixing` on strict clippy pedantic
without introducing new suppression attributes.

## Why

After `xiuxian-daochang` convergence, cross-crate pedantic runs still surfaced
quality debt in:

- qianhuan hot-reload/manifestation production paths,
- qianhuan tests with widespread `expect`/`expect_err`,
- zhixing agenda-render return shape and minor test-level style warnings.

## Implemented Changes

### `xiuxian-qianhuan` production code

1. Added missing `# Errors` docs for `Result`-returning API in:
   - `src/hot_reload/backend.rs`
2. Collapsed identical `match` arms in:
   - `src/hot_reload/driver.rs`
3. Replaced manual char comparison with idiomatic array-based containment in:
   - `src/manifestation/manager.rs`
4. Removed `unused_self` pattern by converting a helper to associated function:
   - `src/manifestation/templates.rs`

### `xiuxian-qianhuan` tests

1. Eliminated `expect`/`expect_err` usage in:
   - `tests/test_dynamic_template_loading.rs`
   - `tests/test_manifestation_manager.rs`
   - `tests/unit_persona.rs`
2. Standardized tests to `Result`-based flow with `?` and explicit failure
   branches.
3. Removed needless raw-string hashes where not required.
4. Replaced `Default::default()` with explicit `HashMap::default()` in test
   runtime context payload.

### `xiuxian-zhixing`

1. Removed unnecessary `Result` wrapping from agenda note-render helper:
   - `src/heyi/agenda_render.rs`
2. Applied idiomatic `?` in `Option` flow where clippy suggested.
3. Replaced direct float equality assertion with epsilon-based check:
   - `tests/test_agenda_entry.rs`
4. Replaced unit-struct `default()` call with direct construction:
   - `tests/test_strict_teacher.rs`

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-qianhuan -p xiuxian-zhixing
cargo clippy -p xiuxian-qianhuan --all-targets -- -W clippy::pedantic
cargo clippy -p xiuxian-zhixing --all-targets -- -W clippy::pedantic
cargo test -p xiuxian-qianhuan --test test_dynamic_template_loading
cargo test -p xiuxian-qianhuan --test test_manifestation_manager
cargo test -p xiuxian-qianhuan --test unit_persona
cargo test -p xiuxian-zhixing --test test_agenda_entry
cargo test -p xiuxian-zhixing --test test_strict_teacher
```

Result:

- Both clippy lanes passed under pedantic for all targets.
- `xiuxian-qianhuan` targeted tests passed (`3 + 5 + 4` tests).
- `xiuxian-zhixing` targeted tests passed (`1 + 1` tests).

## Outcome

This wave removed real quality debt in two crates without suppression-based
shortcuts, and established `Result`-first test style consistency in
`xiuxian-qianhuan` high-impact test modules.
