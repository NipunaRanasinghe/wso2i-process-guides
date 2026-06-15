# Release Process

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/10 \
_Updated_: 2026/06/12

This document describes the manual process for deciding when and how to create a release. The goal is that any release manager can complete end-to-end releases with the help of this guide.

> For detailed information about the automated pipelines and configuration, see [CI/CD Pipelines](../cicd-pipelines.md).

## Release Schedule

- **Nightly**: a new build is produced automatically on a daily schedule.
- **Stable / GA**: feature releases target quarterly; patch releases target every 2 weeks. Hotfixes ship immediately for critical issues. The release manager decides based on feature readiness and the stability of the nightly builds.

## Release Ownership

Each stable release has a designated release manager — typically a rotation. The release manager is responsible for the overall delivery of the release, including:

- Creating and managing the release milestones
- Triggering the release workflows
- Coordinating the team verification and documentation efforts
- Communicating release status and updates to the team

## Release Types

There are three types of stable releases. All three ship through the same Stable/GA release pipeline (see [Release Pipelines](../cicd-pipelines.md#release-pipelines)) — they differ in source branch, scope, and urgency.

| Type | Version Bump | Frequency |
|---|---|---|
| **Feature** | Minor | Quarterly, when planned features are ready |
| **Patch** | Patch | Every 2 weeks, as bug fixes accumulate on the maintenance branch |
| **Hotfix** | Patch | On-demand, for critical issues that cannot wait for the patch cycle |

The branch mechanics behind each type are defined in [Branching Strategy](../branching-strategy.md).

> **Note:** For `product-integrator`, the released version is the WSO2 Integrator product version — managed at the product level rather than bumped mechanically (see [Product Distribution Versioning](../versioning-strategy.md#product-distribution-versioning)).

## Release Order

A WSO2 Integrator stable release spans multiple repos. Because cross-repo dependencies are pinned (see [Cross-Repo Version Propagation](../versioning-strategy.md#cross-repo-version-propagation)), the repos _must_ be prepared and released in dependency order:

1. **Shared UI libraries** — not released separately. Before a product release, each product repo updates its shared-libraries submodule pointer to the commit it intends to ship; the libraries are built from source within the product build.
2. **Product extensions** — release `ballerina-tooling`, `mi-tooling`, and `si-tooling`, each through its own pipeline and approval gate.
3. **Product distribution** — bump the pinned extension versions in `product-integrator`, then trigger its release.

A repo with no changes since its last release does not need to release — the existing pins continue to resolve.

## Release Processes

Each release type has its own step-by-step guide:

- [Feature Release Guide](feature-release.md)
- [Patch Release Guide](patch-release.md)
- [Hotfix Release Guide](hotfix-release.md)

## Pending Items

The following items represent gaps between this proposal and the current state of the repos.

- **Implement the automated nightly publish pipeline.** The continuous nightly publish described in Release Schedule does not yet exist. Merges to `main` do not trigger a publish in any repo. The pipeline needs to be built before the schedule can be followed.
