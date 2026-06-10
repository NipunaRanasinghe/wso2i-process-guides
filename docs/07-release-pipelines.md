# Release Pipelines & Channels

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/10

This document describes the two-track release model, artifact publishing targets, and the approval gate applied to Stable/GA releases.

## Two-Track Model

```
main branch
    │
    ├──► Nightly build (automated, on every merge to main)
    │         └─► Publish to "Insider" VS Code Marketplace channel
    │             Bundled into WSO2 Integrator IDE Insider build
    │             Published to GitHub Releases as a pre-release tag
    │
    └──► Stable/GA release (via release workflow + approval gate)
              └─► Publish to VS Code Marketplace (stable channel)
                  Bundled into WSO2 Integrator IDE Stable build
                  Published to https://github.com/wso2/product-integrator/releases
```

## Nightly / Insider Channel

- Triggered by every merge to `main` in any product repo.
- Versions are suffixed: `1.2.0-nightly.20260609`.
- Published automatically with no approval gate.
- Serves as a preview channel for early adopters and QA.

## Stable / GA Channel

- Triggered by a manually dispatched release workflow targeting a specific commit on `main` or the `<major>.<minor>.x` maintenance branch.
- Publishes clean SemVer tags (`1.2.0`).
- Requires an approval gate before the VS Code Marketplace publish step (see [Approval Gates](#approval-gates) below).
- The `product-integrator` GA bundle _must_ be published to [GitHub Releases](https://github.com/wso2/product-integrator/releases) as the final step.

## Artifact Publishing Targets

| Artifact | Nightly Channel | Stable Channel |
|---|---|---|
| VS Code extensions (×3 products) | VS Code Marketplace (pre-release) | VS Code Marketplace (stable) |
| WSO2 Integrator IDE | GitHub Releases (pre-release tag) | GitHub Releases (stable tag) |

## Approval Gates

- **Nightly:** fully automated — no approval needed. Speed and frequency are the point.
- **GA:** a GitHub Actions [Environment](https://docs.github.com/en/actions/deployment/targeting-different-environments) named `production` _must_ be configured with required reviewers (1–2 people). The publish step targets this environment, pausing until a reviewer approves in the GitHub UI. This ensures a human signs off before every public stable release.
