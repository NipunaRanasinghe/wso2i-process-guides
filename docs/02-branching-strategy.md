# Branching Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/11

This document defines the branching model for all WSO2 Integrator repos — the shared UI toolkit, product tooling, and product distribution layers.

## Rationale

The WSO2 Integrator tooling spans five repos with contributors of varying experience levels and an increasing use of agentic development workflows. The branching model _must_ be simple enough to apply consistently across all repos while still supporting parallel patch maintenance on stable releases.

### Considered Options

- **Trunk-Based Development** — all commits go directly (or via very short branches) to `main`.
- **GitFlow** — parallel `main`/`develop` branches with dedicated release branches.
- **GitHub Flow with Maintenance Branch** — a single `main` branch for features and a dedicated `<major>.<minor>.x` branch for patches.

### Selected Option: GitHub Flow with Maintenance Branch

- Avoids the overhead of managing multiple long-lived branches (e.g. `develop`) and complex merge patterns.
- Supports a clear separation of concerns between feature development and patch maintenance.
- Aligns well with the release process defined in [Release Process](07-release-process.md) and the versioning strategy in [Versioning Strategy](03-versioning-strategy.md).

## Branches

### The `main` Branch

The primary integration branch. `main` _must_ always be in a releasable state and serves as the base for:

- The next feature release (often a minor version bump)
- Nightly builds
- Milestone releases (e.g. `5.1.0-m1`) ahead of the next feature release

### Feature Branches

All feature development _must_ happen on a dedicated feature branch. Branch names _must_ follow the `feat/<description>` convention (e.g. `feat/workflow-support`).

- Feature branches _must_ be merged to `main` only when the feature is stable and release-ready.
- Feature branches _must_ be deleted after merging.

### Patch Branch (`<major>.<minor>.x`)

Only one active patch branch exists at any given time (e.g. `5.0.x`). This branch _must_ always be release-ready and serves as the base for all patch releases (e.g. `5.0.1`, `5.0.2`).

- All bug fixes _must_ be submitted to the active patch branch — not to `main`.
- Repo maintainers _should_ merge the active patch branch into `main` promptly.
- When a new minor GA is released, the previous patch branch is retired and a new one is created from the new GA tag.

### Hotfix Branches

Hotfix branches are used for critical issues that require an immediate patch release and cannot wait for the normal bug fix cycle.

- Hotfix branches _must_ be created from the latest stable release tag and follow the `hotfix/<description>` naming convention (e.g. `hotfix/critical-auth-bypass`).
- Once the fix is released, hotfix branches _must_ be merged back into the active patch branch.
- Repo maintainers _should_ ensure hotfixes are also merged into `main` if they apply to the current development version.
