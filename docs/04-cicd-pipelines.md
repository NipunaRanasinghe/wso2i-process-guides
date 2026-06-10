# CI/CD Pipelines

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/11

This document describes the GitHub Actions pipeline anatomy for pull requests and merges to `main` across all repos.

**Platform:** GitHub Actions. **Build tools:** Gradle (language servers), Rush (TS extensions).

## PR Pipeline

The PR pipeline runs on every `pull_request` targeting `main` and _must_ pass before any merge is permitted.

```mermaid
graph LR
    T(["pull_request → main"]) --> S1["Compile<br>(Gradle / Rush)"]
    S1 --> S2["Unit tests"]
    S2 --> S3["Integration tests"]
    S3 --> S4["Quality gate"]
    S4 --> S5["Dependency scan<br>(Trivy)"]
```

The quality gate and dependency scan steps are described in [Quality & Security Gates](06-quality-and-security-gates.md).

## Merge-to-Main Pipeline

The merge-to-main pipeline runs on every push to `main` and produces the nightly candidate artifact.

```mermaid
graph LR
    T(["push → main"]) --> S1["All PR steps<br>(re-run on merge commit)"]
    S1 --> S2["Build & package artifact<br>(VSIX / JAR)"]
    S2 --> S3["Publish to Nightly / Insider channel"]
```

The publish step feeds the Nightly / Insider release track (see [Release Pipelines](#release-pipelines)).

## Cross-Repo Coordination

Product repos publish versioned VSIX/package artifacts to a GitHub Packages registry on each merge to `main`. The `product-integrator` bundling pipeline declares explicit dependency versions and is triggered separately — it does not auto-follow upstream `main` commits. This prevents a product-repo commit from inadvertently breaking the IDE build.

## Release Pipelines

All release pipelines run on GitHub Actions. There are two tracks.

```mermaid
graph LR
    M["main branch"] --> N["Nightly / Insider<br>(automated — every merge to main)"]
    M --> S["Stable / GA<br>(manual workflow_dispatch)"]
    N --> NT["VS Code Marketplace (pre-release channel)<br>WSO2 Integrator IDE Insider build<br>GitHub Releases (pre-release tag)"]
    S --> ST["VS Code Marketplace (stable channel)<br>WSO2 Integrator IDE Stable build<br>GitHub Releases (stable tag)"]
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
| VS Code extensions (×4) | VS Code Marketplace (pre-release) | VS Code Marketplace (stable) |
| WSO2 Integrator IDE | GitHub Releases (pre-release tag) | GitHub Releases (stable tag) |
