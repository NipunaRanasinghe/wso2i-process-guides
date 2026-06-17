# Branching Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/11

This document defines the branching model for all WSO2 Integrator repos: the shared UI libraries, product tooling, and product distribution layers.

## Rationale

The WSO2 Integrator tooling spans five repos with contributors of varying experience levels and an increasing use of agentic development workflows. The branching model _must_ be simple enough to apply consistently across all repos while still supporting parallel patch maintenance on stable releases.

### Considered Options

- **Trunk-Based Development** — all commits go directly (or via very short branches) to `main`.
- **GitFlow** — parallel `main`/`develop` branches with dedicated release branches.
- **GitHub Flow with Maintenance Branch** — a single `main` branch for features and a dedicated `<major>.<minor>.x` branch for patches.

### Selected Option: GitHub Flow with Maintenance Branch

- Avoids the overhead of managing multiple long-lived branches (e.g. `develop`) and complex merge patterns.
- Supports a clear separation of concerns between feature development and patch maintenance.
- Aligns well with the release process defined in [Release Process](release-process/) and the versioning strategy in [Versioning Strategy](versioning-strategy.md).

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

A patch branch tracks all bug fixes and security patches for a specific released minor version. Each patch branch _must_ always be release-ready and serves as the base for all patch releases in that version line (e.g. `5.0.1`, `5.0.2`). The number of active patch branches differs by repo type.

**Product tooling repos** (`ballerina-tooling`, `mi-tooling`, `si-tooling`): Only one active patch branch exists at any given time — the branch for the latest stable minor version (e.g. `1.2.x`). VS Code extensions do not backport fixes to older minor version lines. When a new minor GA is released, the previous patch branch is retired and a new one is created from the new GA tag.

**Product distribution repo** (`product-integrator`): Multiple patch branches may be active simultaneously, one per supported minor version (e.g. `5.0.x` and `5.1.x`). Because the WSO2 Integrator IDE follows product EoL policies, critical fixes may need to be backported to older minor versions that are still within their support window. A patch branch is retired when its minor version reaches end of life.

Across all repos:

- Bug fixes and security patches _must_ be submitted to the relevant active patch branch, not to `main`.
- Repo maintainers _should_ merge each active patch branch into `main` promptly after a patch release.

### Hotfix Branches

Hotfix branches are used for critical issues that require an immediate patch release and cannot wait for the normal bug fix cycle.

- Hotfix branches _must_ be created from the latest stable release tag and follow the `hotfix/<description>` naming convention (e.g. `hotfix/critical-auth-bypass`).
- Once the fix is released, hotfix branches _must_ be merged back into the active patch branch.
- Repo maintainers _should_ ensure hotfixes are also merged into `main` if they apply to the current development version.

## Pending Items

The following items represent gaps between this proposal and the current state of the repos.

- **Align release branch naming in `ballerina-tooling` and `mi-tooling`:** These repos currently use `stable/ballerina` and `stable/mi` as their release branches, not the `<major>.<minor>.x` convention defined here. Aligning branch naming requires coordination with the ongoing release cycles in both repos.
