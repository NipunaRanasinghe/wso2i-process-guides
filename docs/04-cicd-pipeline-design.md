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
│  4. Quality gate     (see Quality Gates)                        │
│  5. Dependency scan  (see Quality Gates)                        │
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
│  3. Publish to Nightly/Insider channel (see Release Pipelines ↓)│
└─────────────────────────────────────────────────────────────────┘
```

## Cross-Repo Coordination

Product repos publish versioned VSIX/package artifacts to a GitHub Packages registry on each merge to `main`. The `product-integrator` bundling pipeline declares explicit dependency versions and is triggered separately — it does not auto-follow upstream `main` commits. This prevents a product-repo commit from inadvertently breaking the IDE build.

## Release Pipelines

All release pipelines run on GitHub Actions. There are two tracks.

```
main branch
    │
    ├──► Nightly / Insider  (automated — every merge to main)
    │         └─► VS Code Marketplace (pre-release channel)
    │             WSO2 Integrator IDE Insider build
    │             GitHub Releases (pre-release tag)
    │
    └──► Stable / GA  (manually dispatched workflow_dispatch)
              └─► VS Code Marketplace (stable channel)
                  WSO2 Integrator IDE Stable build
                  GitHub Releases (stable tag) — https://github.com/wso2/product-integrator/releases
```

### Nightly / Insider Pipeline

- Triggered automatically on every merge to `main`.
- Version suffix: `1.2.0-nightly.20260609`.
- Fully automated — no approval gate.

### Stable / GA Pipeline

- Triggered by a manually dispatched `workflow_dispatch` targeting a specific commit on `main` or the `<major>.<minor>.x` maintenance branch.
- Publishes clean SemVer tags (e.g. `1.2.0`).
- The publish step targets a GitHub Actions [Environment](https://docs.github.com/en/actions/deployment/targeting-different-environments) named `production`, configured with 1–2 required reviewers. The workflow pauses here until a reviewer approves in the GitHub UI.

### Artifact Publishing Targets

| Artifact | Nightly | Stable |
|---|---|---|
| VS Code extensions (×3 products) | VS Code Marketplace (pre-release) | VS Code Marketplace (stable) |
| WSO2 Integrator IDE | GitHub Releases (pre-release tag) | GitHub Releases (stable tag) |
