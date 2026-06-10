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
| Tooling / UI E2E tests | Nightly schedule + manual trigger | **No** (reported async) | Requires a full runtime environment; too slow for the PR feedback loop |
| Backward compatibility tests | Release pipeline (Stable/GA) | **Yes** | Run against the previous GA release artifact before a new GA ships |

> **Note:** Adding a `run-e2e` label to a PR triggers E2E as advisory feedback without blocking merge. This is _recommended_ for high-risk changes.
