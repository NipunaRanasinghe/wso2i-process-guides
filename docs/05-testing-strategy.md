# Testing Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/14

This document defines the test types integrated into the CI/CD pipelines, their placement in the pipeline stages, and which gates are blocking.

## Approach

The tooling follows a layered testing model — a test pyramid. Most coverage sits at the base in fast unit tests; a smaller layer of integration tests checks that components work together; and a thin top layer of end-to-end (E2E) tests drives the full product through its UI. A separate backward-compatibility check runs at release time against the previous GA build.

The layers differ in two ways: how fast they run, and how much of the real product they exercise. Unit tests run modules in isolation and finish quickly, so they run on every pull request and catch most regressions early. Integration tests are slower — they launch the packaged language server — but they catch the one thing unit tests cannot: a mismatch between an extension and the language server it ships with. E2E tests run the real product in a full VS Code environment, which is the most thorough check and the slowest, so running them on every change is not practical.

This is why the tests are split this way. Catching each bug at the lowest layer that can catch it keeps the per-PR checks fast enough to block every merge, while the slow E2E runs move to a daily schedule instead of gating each PR. The team still gets coverage across the whole stack, without running the full E2E suite on every commit.

## Test Matrix

| Test Type | Applies To | Runs In | Blocks Merge? | Notes |
|---|---|---|---|---|
| Unit tests | All components | PR pipeline + daily build | **Yes** | _Must_ pass before any artifact is published |
| Integration tests | VS Code extension + language server | PR pipeline + daily build | **Yes** | Tests extension ↔ language server contract |
| Tooling / UI E2E tests | VS Code extensions | PR (label: `Checks/Run Ballerina UI Tests`) · daily schedule · manual trigger | **No** (advisory) | `ballerina-tooling` only currently. Requires a full runtime environment; too slow to run on every PR. |
| Backward compatibility tests | Product distribution (GA artifact) | Release pipeline (Stable/GA) | **Yes** | Run against the previous GA release artifact before a new GA ships |

## Test Levels

### Unit Tests

Unit tests validate individual modules in isolation — Gradle test suites for the language servers and the Rush test task for the TypeScript packages. They run on every PR and every daily build, and are merge-blocking: the PR author _must_ fix failures before the PR can be merged.

### Integration Tests

Integration tests validate the contract between the extension and its bundled language server: the test harness launches the packaged language server and invokes the LSP requests the extension depends on (initialization, completions, diagnostics, code actions). This catches mismatches between the extension and the language server that are released together, which unit tests on either side cannot. Integration tests run on every PR and every daily build, and are merge-blocking.

### Tooling / UI E2E Tests

E2E tests drive the full VS Code UI against a real runtime environment, covering user-facing workflows end to end. They are too slow to run on every PR and require a full runtime environment, so they run on a daily schedule against `main`, on demand via manual trigger, and on a PR when the `Checks/Run Ballerina UI Tests` label is applied. Authors of UI-affecting changes _should_ apply the label before requesting review.

E2E results are advisory — a failure does not block a merge — but failures of the daily run _must_ be triaged by the repo maintainers before the next stable release.

> **Note:** E2E coverage currently exists only in `ballerina-tooling`. 

## Pending Items

The following items represent gaps between this proposal and the current state of the repos.

- **No integration test step exists in any repo.** The "extension ↔ language server contract" integration tests described here are not yet implemented. The test infrastructure needs to be defined and added to the PR pipelines of `ballerina-tooling`, `mi-tooling`, and `si-tooling`.
- **Backward compatibility tests not implemented.** No repo has backward compatibility tests in its release pipeline. This gate cannot block a release until the tests are built and wired in.
- **`mi-tooling` already has Playwright E2E tests.** The note above is outdated — `mi-tooling` has a `UITest` job with 4 parallel groups running on both Linux and Windows. The test matrix table should be updated to reflect this once E2E coverage is audited across all repos.
- **No merge-to-main test run.** No repo has a `push` trigger on `main`. The test matrix column "Runs In: PR pipeline + merge-to-main" overstates current coverage; merge coverage is provided only by the daily cron builds.
