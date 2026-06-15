# Testing Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/15

This document defines the test types integrated into the CI/CD pipelines, their placement in the pipeline stages, and which gates are blocking.

## Approach

The tooling follows the standard test pyramid: unit tests at the base, component tests above them, integration tests above that, and E2E tests at the top. A smoke test layer covers the full product installer.

## Test Matrix

| Test Type | Applies To | Runs In | Blocks Merge? |
|---|---|---|---|
| Unit tests | All components | PR pipeline + daily build | **Yes** |
| Component tests | VS Code extensions | PR pipeline | **Yes** |
| Integration tests | VS Code extension + language server | PR pipeline + daily build | **Yes** |
| E2E tests | VS Code extensions | PR pipeline (on demand) + daily build | **No** (advisory) |
| Smoke tests | Product distribution (IDE installer) | Release pipeline (pre-release + stable) | **Yes** (`hello-world-service`) / **No** (`icp`) |

## Test Levels

### Unit Tests

Unit tests validate individual modules in isolation.

- **Language servers** (Gradle / JUnit): `./gradlew test` covers the Ballerina language server logic. Runs on the daily build targeting both `main` and the latest stable branch.
- **TypeScript packages** (Rush / pnpm): `rush test` covers extension utility code and shared packages.

### Component Tests

Component tests use Jest with React Testing Library to validate the rendering correctness of individual UI components. Each test renders a component via `react-test-renderer` and compares the output against a stored baseline snapshot; a diff fails the test. They run on PRs when relevant source files change (path-filter gated) and on the daily build.

- **`ballerina-tooling`:** `bi-diagram`, `component-diagram`, `type-diagram`, `sequence-diagram`
- **`mi-tooling`:** `mi-diagram`

### Integration Tests

Integration tests validate the contract between the extension and its bundled language server. The test harness launches the packaged language server and invokes the LSP requests the extension depends on: initialization, completions, diagnostics, code actions. This catches mismatches between the extension and the language server that unit tests on either side cannot catch.

### E2E Tests

E2E tests use Playwright to drive the full VS Code UI against a real runtime environment, covering user-facing workflows end to end. They are too slow to run on every PR and require a full runtime environment, so they run on a daily schedule against `main`, on demand via manual trigger, and on a PR when the relevant label is applied.

- **`ballerina-tooling`:** 4 parallel groups on Linux. Label: `Checks/Run Ballerina UI Tests`.
- **`mi-tooling`:** 4 parallel groups on Linux and 4 on Windows (8 jobs total). Label: `Checks/Run MI UI Tests` or `Checks/Enable UI Tests`.

Authors of UI-affecting changes _should_ apply the label before requesting review.

E2E results are advisory: a failure does not block a merge. Failures of the daily run _must_ be triaged by the repo maintainers before the next stable release.

### Smoke Tests

Smoke tests drive the full WSO2 Integrator IDE installer end-to-end using `wso2ipw`, the Integrator automation tool. They run on both Linux and Windows against the built installer artifact during the release pipeline.

- **`hello-world-service`:** Blocking — the release pipeline does not proceed if this test fails.
- **`icp`:** Advisory — failures are reported but do not block the release.

Smoke tests apply only to `product-integrator`. They can also be triggered manually against a published GitHub Release artifact for regression testing.

## Pending Items

The following items represent gaps between this proposal and the current state of the repos.

- **Implement integration tests in `ballerina-tooling`, `mi-tooling`, and `si-tooling`.** The "extension ↔ language server contract" integration tests described here are not yet implemented. The test infrastructure needs to be defined and added to the PR pipelines of each repo.
- **Add language server unit tests as a required PR gate.** Language server unit tests currently run only in the daily build and need to be added as a blocking PR gate.
- **Re-enable `ExtTest_Ballerina` in `ballerina-tooling`.** The extension unit test job has `if: false` pending test stability improvements. Extension unit tests do not currently run in CI.
- **Add test coverage to `si-tooling`.** The `build.yml` workflow contains no test step — build and package only. Unit tests, E2E tests, and Trivy scanning all need to be added.
