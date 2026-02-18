# Operating Checklists

Use these checklists for day-to-day execution of this modernization playbook.

## 1. Daily Development Checklist

1. Confirm which feature name from the roadmap you are executing.
2. Verify code ownership and boundary impact (Rust-only, Python-only, cross-language).
3. Run focused tests before broad tests.
4. Capture evidence:
   - commands run,
   - pass/fail outcomes,
   - changed contracts.
5. Update related playbook files when assumptions changed.

## 2. Rust PR Checklist

1. Crate manifest includes `[lints] workspace = true`.
2. No unnecessary `allow` lints introduced.
3. No accidental stdout/stderr output path introduced in protocol/runtime libraries.
4. Tests added or updated for behavior changes.
5. New public APIs have typed error behavior and documentation notes.

## 3. Python PR Checklist

1. Module responsibility is clear and not duplicated across packages.
2. Boundary payload changes are covered by tests.
3. Large-file edits include modularization consideration.
4. Config/path handling uses PRJ conventions.

## 4. Cross-Language Boundary Checklist

1. Contract shape is explicit (JSON schema, typed model, or IPC schema).
2. Rust and Python sides share the same semantic meaning for fields.
3. Failure behavior is documented and tested.
4. Backward compatibility impact is declared.

## 5. CI Checklist

1. Lint lane passes.
2. Build lane passes.
3. Test lane passes (including targeted contract/perf tests).
4. Dependency security lane passes.
5. Required result-gatherer job reports success.

## 6. Release Checklist

1. Version and tag consistency verified.
2. Target platform build set verified.
3. Artifact integrity checks completed.
4. Rollback path tested/documented.
5. Release note references features (not vague phase labels).

## 7. Documentation Checklist

1. Feature-level summary updated in roadmap document.
2. If behavior changed, update relevant crate/package docs.
3. Add evidence links and command outputs for non-trivial changes.
