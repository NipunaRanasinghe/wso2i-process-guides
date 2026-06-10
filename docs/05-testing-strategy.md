# Testing Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/11

This document defines the test types integrated into the CI/CD pipelines, their placement in the pipeline stages, and which gates are blocking. 

## Test Matrix

| Test Type | Runs In | Blocks Merge? | Notes |
|---|---|---|---|
| Unit tests | PR pipeline + merge-to-main | **Yes** | _Must_ pass before any artifact is published |
| Integration tests | PR pipeline + merge-to-main | **Yes** | Tests extension ↔ language server contract |
| Tooling / UI E2E tests | PR (label: `Checks/Run Ballerina UI Tests`) · daily schedule · manual trigger | **No** (advisory) | `ballerina-tooling` only currently. Requires a full runtime environment; too slow to run on every PR. |
| Backward compatibility tests | Release pipeline (Stable/GA) | **Yes** | Run against the previous GA release artifact before a new GA ships |

## Test Levels

### Unit Tests

Unit tests validate individual modules in isolation — Gradle test suites for the language servers and the Rush test task for the TypeScript packages. They run on every PR and every merge to `main`, and are merge-blocking: the PR author _must_ fix failures before the PR can be merged.

### Integration Tests

Integration tests validate the contract between the extension and its bundled language server: the test harness launches the packaged language server and exercises the LSP requests the extension depends on (initialization, completions, diagnostics, code actions). This catches mismatches between the two halves of a release pair, which unit tests on either side cannot. Integration tests run on every PR and every merge to `main`, and are merge-blocking.

### Tooling / UI E2E Tests

E2E tests drive the full VS Code UI against a real runtime environment, covering user-facing workflows end to end. They are too slow and environment-heavy to run on every PR, so they run on a daily schedule against `main`, on demand via manual trigger, and on a PR when the `Checks/Run Ballerina UI Tests` label is applied. Authors of UI-affecting changes _should_ apply the label before requesting review.

E2E results are advisory — a failure does not block a merge — but failures of the daily run _must_ be triaged by the repo maintainers before the next stable release.

> **Note:** E2E coverage currently exists only in `ballerina-tooling`. 
