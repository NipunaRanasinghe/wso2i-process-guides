# Testing Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/10

This document defines the test types integrated into the CI/CD pipelines, their placement in the pipeline stages, and which gates are blocking.

## Test Matrix

| Test Type | Runs In | Blocks Merge? | Notes |
|---|---|---|---|
| Unit tests | PR pipeline + merge-to-main | **Yes** | _Must_ pass before any artifact is published |
| Integration tests | PR pipeline + merge-to-main | **Yes** | Tests extension ↔ language server contract |
| Tooling / UI E2E tests | PR (label: `Checks/Run Ballerina UI Tests`) · daily schedule · manual trigger | **No** (advisory) | `ballerina-tooling` only currently. Requires a full runtime environment; too slow to run on every PR. |
| Backward compatibility tests | Release pipeline (Stable/GA) | **Yes** | Run against the previous GA release artifact before a new GA ships |
