# Release Process

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/10 \
_Updated_: 2026/06/12

This document describes the manual process for deciding when and how to create a release. The goal is that any release owner can complete end-to-end releases with the help of this guide. 

> For detailed information about the automated pipelines and configuration, see [CI/CD Pipelines](04-cicd-pipelines.md).

## Table of Contents

- [Release Cadence](#release-cadence)
- [Release Ownership](#release-ownership)
- [Release Types](#release-types)
- [Release Order](#release-order)
- [Feature Release Process](#feature-release-process)
- [Patch Release Process](#patch-release-process)
- [Hotfix Release Process](#hotfix-release-process)

## Release Cadence

- **Nightly**: continuous — a new nightly build is produced automatically on a daily schedule.
- **Stable / GA**: target every 4–6 weeks, or immediately for critical patch fixes. The release owner decides based on feature readiness and the stability of the nightly builds.

## Release Ownership

Each stable release has a designated release owner — typically a rotation. The release owner is responsible for the overall delivery of the release, including:

- Creating and managing the release milestone
- Triggering the release workflow
- Obtaining the approval-gate sign-off
- Completing the post-release steps

## Release Types

There are three types of stable releases. All three ship through the same Stable/GA release pipeline (see [Release Pipelines](04-cicd-pipelines.md#release-pipelines)) — they differ in source branch, scope, and urgency.

| Type | Source Branch | Version Bump | When |
|---|---|---|---|
| **Feature** | `main` | Minor (or major) | Every 4–6 weeks, when planned features are ready |
| **Patch** | `<major>.<minor>.x` | Patch | As bug fixes accumulate on the maintenance branch |
| **Hotfix** | `hotfix/<description>`, created from the latest stable tag | Patch | Immediately, for critical issues that cannot wait for the patch cycle |

The branch mechanics behind each type are defined in [Branching Strategy](02-branching-strategy.md).

> **Note:** For `product-integrator`, the released version is the WSO2 Integrator product version — managed at the product level rather than bumped mechanically (see [Product Distribution Versioning](03-versioning-strategy.md#product-distribution-versioning)).

## Release Order

A WSO2 Integrator stable release spans multiple repos. Because cross-repo dependencies are pinned (see [Cross-Repo Version Bumps](03-versioning-strategy.md#cross-repo-version-bumps)), the repos _must_ be prepared and released in dependency order:

1. **Shared UI libraries** — not released separately. Before a product release, each product repo updates its shared-libraries submodule pointer to the commit it intends to ship; the libraries are built from source within the product build.
2. **Product extensions** — release `ballerina-tooling`, `mi-tooling`, and `si-tooling`, each through its own pipeline and approval gate.
3. **Product distribution** — bump the pinned extension versions in `product-integrator`, then trigger its release.

A repo with no changes since its last release does not need to release — the existing pins continue to resolve.

## Feature Release Process

### Step 1: Create the Release Milestone

The release cycle starts when the release owner creates a dedicated milestone in the `product-integrator` repo (e.g. [WSO2 Integrator 5.1.0](https://github.com/wso2/product-integrator/milestone/7)), if one does not already exist. All features and fixes planned for the release are tracked against this milestone.

### Step 2: Pre-Release Verification

Before triggering the release workflow, the release owner verifies:

- [ ] All issues and PRs in the release milestone are closed, or moved to a future milestone
- [ ] All planned features for this release are merged to `main`
- [ ] No critical open issues or regressions in the nightly pre-release channel
- [ ] Changelog updated for the release version

During this window the release owner _should_ announce a code freeze on `main` branch.

### Step 3: Trigger the Release Workflow

1. Go to the **Actions** tab of the target repo and select the stable release workflow (`release.yml`).
2. Click **Run workflow** and enter the target commit SHA on `main`.
3. The workflow builds and packages artifacts, runs the backward compatibility tests against the previous GA release, then pauses at the `production` environment gate.

### Step 4: Approval Gate

A required reviewer inspects the run and approves in the GitHub UI — the publish step then proceeds. If the reviewer rejects, the reviewer _must_ state the reason on the workflow run; the release owner addresses it and re-triggers.

### Step 5: Post-Release Steps

After the GA artifacts are published:

1. **Verify the tag** — confirm `v<major>.<minor>.<patch>` was created on the release commit.
2. **Create the maintenance branch** — create `<major>.<minor>.x` from the GA tag, and retire the previous maintenance branch (no further releases are created from it; the branch is kept for history).
3. **Bump `main`** — open a PR on `main` incrementing to the next minor dev version (e.g. `1.3.0-dev`).
4. **Confirm the GitHub Release** — verify the `product-integrator` bundle is published to [GitHub Releases](https://github.com/wso2/product-integrator/releases) with release notes.
5. **Update the milestones** — close the release milestone and create the milestone for the next immediate patch release (e.g. `5.1.1`).
6. **Communicate** — notify the team and update any public-facing changelogs or announcements.

## Patch Release Process

A patch release targets the `<major>.<minor>.x` maintenance branch, not `main`.

### Step 1: Confirm the Release Milestone

Confirm the dedicated milestone for this patch release exists in the `product-integrator` repo (e.g. [WSO2 Integrator 5.0.1](https://github.com/wso2/product-integrator/milestone/8)), and create it if it does not. All changes shipping in the release are tracked against this milestone.

### Step 2: Qualify the Fixes

Confirm every change qualifies — bug fixes and security patches only; no new features.

### Step 3: Merge to the Maintenance Branch

Open a PR against `<major>.<minor>.x` for each fix. All PR pipeline gates _must_ pass.

### Step 4: Trigger the Release Workflow

Trigger the stable release workflow targeting `<major>.<minor>.x`. The approval gate applies as in the feature release ([Step 4](#step-4-approval-gate)).

### Step 5: Post-Release Steps

1. **Verify the tag** — confirm `v<major>.<minor>.<patch>` was created on the release commit.
2. **Confirm the GitHub Release** — verify artifacts and release notes are published.
3. **Merge to `main`** — merge the maintenance branch into `main` so the fixes are included in the next feature release.
4. **Update the milestones** — close the release milestone and create the milestone for the next immediate patch release (e.g. `5.0.2`).
5. **Communicate** — notify the team; for security fixes, describe the fix without exposing exploit details.

## Hotfix Release Process

A hotfix release is for critical issues — security vulnerabilities, data loss, a broken release — that cannot wait for the normal patch cycle. Unlike a patch release, a hotfix isolates the critical fix on top of exactly what users are running — use it when the maintenance branch contains other unreleased changes that cannot be validated in time.

### Step 1: Create the Hotfix Branch

Create `hotfix/<description>` from the latest stable release tag (e.g. `hotfix/critical-auth-bypass` from `v5.0.2`).

### Step 2: Apply and Verify the Fix

Open a PR against the hotfix branch with the minimal fix. All PR pipeline gates _must_ pass — the urgency does not waive any gate.

### Step 3: Trigger the Release Workflow

Trigger the stable release workflow targeting the hotfix branch. The release owner _should_ notify the approval-gate reviewers in advance so the gate does not delay the release.

The hotfix takes the next patch version (e.g. `5.0.3` on top of `v5.0.2`); after the merge-back, the maintenance branch continues from that version.

### Step 4: Verify the Release

Confirm the `v<major>.<minor>.<patch>` tag was created on the release commit, and the artifacts and release notes are published.

### Step 5: Merge Back

1. Merge the hotfix branch into the active `<major>.<minor>.x` maintenance branch.
2. Merge the fix into `main` if it applies to the current development version.
3. Delete the hotfix branch.

### Step 6: Communicate

Notify the team and affected users. For security fixes, publish an advisory without exposing exploit details.

## Pending Items

The following items represent gaps between this proposal and the current state of the repos.

- **No automated Nightly publish.** The continuous nightly publish described in Release Cadence does not yet exist. Merges to `main` do not trigger a publish in any repo. The pipeline needs to be built before the cadence can be followed.
- **No `production` Environment approval gate.** The GitHub Actions Environment with required reviewers does not exist in any repo. Step 4 of the feature and patch release processes cannot be followed until this is configured. Currently, the practical gate is a two-step manual `workflow_dispatch` pattern: a release owner runs `release-vsix.yml` to build and create a draft release, then separately runs `publish-vsix.yml` to publish.
- **Backward compatibility tests not implemented.** Step 3 of the feature release process references backward compatibility tests that do not yet exist. The gate cannot block a release until the tests are built.
- **Release workflow names differ by repo.** The product tooling repos (`ballerina-tooling`, `mi-tooling`, `vscode-extensions`) use `release-vsix.yml` + `publish-vsix.yml` rather than a single `release.yml`. The step-by-step instructions in this document should be updated to reference actual workflow names once the pipeline structure is stabilised.
