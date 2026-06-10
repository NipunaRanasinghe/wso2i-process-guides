# CI/CD Pipeline Design

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/10

This document describes the GitHub Actions pipeline anatomy for pull requests and merges to `main` across all product repos.

**Platform:** GitHub Actions. **Build tools:** Gradle (language servers), Rush (TS extensions).

## PR Pipeline

The PR pipeline _must_ pass before any merge is permitted.

```
┌─────────────────────────────────────────────────────────────────┐
│  On: pull_request → main                                        │
├─────────────────────────────────────────────────────────────────┤
│  1. Compile          (Gradle / Rush build)                      │
│  2. Unit tests       (blocking)                                 │
│  3. Integration tests(blocking)                                 │
│  4. Quality gate     (SonarQube / GHAS — see Quality Gates)     │
│  5. Dependency scan  (Mend / GHAS — see Quality Gates)          │
└─────────────────────────────────────────────────────────────────┘
```

## Merge-to-Main Pipeline

The merge-to-main pipeline runs on every push to `main` and produces the nightly candidate artifact.

```
┌─────────────────────────────────────────────────────────────────┐
│  On: push → main                                                │
├─────────────────────────────────────────────────────────────────┤
│  1. All PR steps (re-run on merge commit)                       │
│  2. Build & package artifact (VSIX / JAR)                       │
│  3. Publish to Nightly/Insider channel (see Release Pipelines)  │
│  4. Trigger E2E tests (non-blocking on this run, reported async)│
└─────────────────────────────────────────────────────────────────┘
```

## Cross-Repo Coordination

Product repos publish versioned VSIX/package artifacts to a GitHub Packages registry on each merge to `main`. The `product-integrator` bundling pipeline declares explicit dependency versions and is triggered separately — it does not auto-follow upstream `main` commits. This prevents a product-repo commit from inadvertently breaking the IDE build.
