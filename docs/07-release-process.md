# Release Process

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/10 \
_Updated_: 2026/06/12

This document describes the manual process for deciding when and how to create a release. The goal is that any release manager can complete end-to-end releases with the help of this guide. 

> For detailed information about the automated pipelines and configuration, see [CI/CD Pipelines](04-cicd-pipelines.md).

## Table of Contents

- [Release Schedule](#release-schedule)
- [Release Ownership](#release-ownership)
- [Release Types](#release-types)
- [Release Order](#release-order)
- [Feature Release Process](#feature-release-process)
- [Patch Release Process](#patch-release-process)
- [Hotfix Release Process](#hotfix-release-process)

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

There are three types of stable releases. All three ship through the same Stable/GA release pipeline (see [Release Pipelines](04-cicd-pipelines.md#release-pipelines)) — they differ in source branch, scope, and urgency.

| Type | Version Bump | When |
|---|---|---|
| **Feature** | Minor | Quarterly, when planned features are ready |
| **Patch** | Patch | Every 2 weeks, as bug fixes accumulate on the maintenance branch |
| **Hotfix** | Patch | On-demand, for critical issues that cannot wait for the patch cycle |

The branch mechanics behind each type are defined in [Branching Strategy](02-branching-strategy.md).

> **Note:** For `product-integrator`, the released version is the WSO2 Integrator product version — managed at the product level rather than bumped mechanically (see [Product Distribution Versioning](03-versioning-strategy.md#product-distribution-versioning)).

## Release Order

A WSO2 Integrator stable release spans multiple repos. Because cross-repo dependencies are pinned (see [Cross-Repo Version Propagation](03-versioning-strategy.md#cross-repo-version-propagation)), the repos _must_ be prepared and released in dependency order:

1. **Shared UI libraries** — not released separately. Before a product release, each product repo updates its shared-libraries submodule pointer to the commit it intends to ship; the libraries are built from source within the product build.
2. **Product extensions** — release `ballerina-tooling`, `mi-tooling`, and `si-tooling`, each through its own pipeline and approval gate.
3. **Product distribution** — bump the pinned extension versions in `product-integrator`, then trigger its release.

A repo with no changes since its last release does not need to release — the existing pins continue to resolve.

## Feature Release Process

### Step 1: Create the Release Milestone

The release cycle starts when the release manager creates a public release milestone in the `product-integrator` repo (e.g. [WSO2 Integrator 5.1.0](https://github.com/wso2/product-integrator/milestone/7)), if one does not already exist. All features and fixes planned for the release must be tracked against this milestone.

### Step 2: Verify the Release Readiness

Before triggering the release workflow, the release manager should verify: 

- [ ] All planned features for this release are completed
- [ ] All issues added in the release milestone are either closed or moved to a future milestone
- [ ] All active patch branches are merged to the main branch (All the bug fixes and security patches gets merged to the active patch branch. See the [Branching-strategy](02-branching-strategy.md#patch-branch-<major>.<minor>.x) documentation for more details)
- [ ] No critical issues or regressions are found in the latest nightly build and the build is stable enough for release

During this window the release manager _should_ announce a code freeze on `main` branch.

### Step 3: Create the Pre-Release Build

The release manager triggers the release workflow targeting the release commit on `main` in pre-release mode. This publishes to the VS Code Marketplace pre-release channel and creates a pre-release tag on GitHub Releases, producing the build that the team _should_ install and verify before the GA build is triggered.

### Step 4: Create and Share the Release Checklist

The release manager creates a GitHub issue in `product-integrator` repo, by listing all PRs included in the release as checkboxes (grouped by component and product team member), and shares the link with the team. 

> Refer to the [release checklist example](https://github.com/wso2/product-ballerina-integrator/issues/2534) for the expected format.


### Step 5: Prepare Release Documentation

The release manager initiates two documentation efforts alongside the team verification. Both _must_ be completed before the GA build ships.

**Release notes:** The release manager drafts release notes covering the changes in this release and shares them with the product manager for review. After the product manager approves, the release manager opens a PR against [wso2/docs-integrator](https://github.com/wso2/docs-integrator). The PR _must_ be merged and the release notes published by the time the GA build goes live.

**Documentation updates:** The release manager coordinates with the product team members responsible for each change to update the [Integrator documentation website](https://github.com/wso2/docs-integrator). All documentation PRs _must_ be reviewed, approved, and merged to the docs repo before the GA release is published.


### Step 6: Team Verification

Each product team member _should_ install the pre-release build, verify their changes, and check off their PRs in the checklist issue.

If a blocker-level issue is found during verification:

1. The fix author merges the fix to `main`.
2. The release manager triggers a new pre-release build from the updated commit (repeat Step 3).
3. The release manager adds a new RC section to the existing checklist issue (e.g. **RC2**) listing the additional PRs, and re-shares the link.
4. The team _should_ verify the new build and check off the new items.

Repeat until no blocker-level issues remain. Non-blocking issues may be deferred to a future release with the release manager's approval. Once all checklist items are verified and no blockers remain, the release manager proceeds to Step 7.

### Step 7: Trigger the Release Build

1. Go to the **Actions** tab of the target repo and select `release-vsix.yml`.
2. Click **Run workflow**, select `main`, and enter the version inputs.
3. The workflow builds the VSIX and creates a draft GitHub Release.

### Step 8: Publish the Release

1. Go to the **Actions** tab and select `publish-vsix.yml`.
2. Click **Run workflow** and enter the run ID from the `release-vsix.yml` run in Step 7.
3. The workflow publishes the VSIX to the VS Code Marketplace and OpenVSX Registry and promotes the draft GitHub Release to published.

### Step 9: Post-Release Steps

After the GA artifacts are published:

1. **Verify the tag** — confirm `v<major>.<minor>.<patch>` was created on the release commit.
2. **Create the maintenance branch** — create `<major>.<minor>.x` from the GA tag, and retire the previous maintenance branch (no further releases are created from it; the branch is kept for history).
3. **Bump `main`** — open a PR on `main` incrementing to the next minor dev version (e.g. `1.3.0-dev`).
4. **Confirm the GitHub Release** — verify the `product-integrator` bundle is published to [GitHub Releases](https://github.com/wso2/product-integrator/releases) with release notes.
5. **Confirm documentation is live** — verify all documentation update PRs and the release notes PR are merged to [wso2/docs-integrator](https://github.com/wso2/docs-integrator) and published on the website.
6. **Update the milestones** — close the release milestone and create the milestone for the next immediate patch release (e.g. `5.1.1`).
7. **Communicate** — notify the team and any affected stakeholders.

## Patch Release Process

A patch release targets the `<major>.<minor>.x` maintenance branch, not `main`.

### Step 1: Confirm the Release Milestone

Confirm the dedicated milestone for this patch release exists in the `product-integrator` repo (e.g. [WSO2 Integrator 5.0.1](https://github.com/wso2/product-integrator/milestone/8)), and create it if it does not. All changes shipping in the release are tracked against this milestone.

### Step 2: Qualify the Fixes

Confirm every change qualifies — bug fixes and security patches only; no new features.

### Step 3: Create the Pre-Release Build

The release manager triggers a pre-release build from the target commit on `<major>.<minor>.x`. This publishes to the VS Code Marketplace pre-release channel and creates a pre-release tag on GitHub Releases, producing the build that the team _should_ install and verify.

### Step 4: Create and Share the Release Checklist

The release manager creates a GitHub issue in `product-integrator` titled `[Release Checklist] WSO2 Integrator <version>` with the `Type/Task` label, listing all PRs included in the release as GFM checkboxes grouped by repo and product team member, and shares the link with the team. Follow the same format as the [feature release checklist](#step-4-create-and-share-the-release-checklist).

### Step 5: Prepare Release Notes

The release manager drafts release notes for the patch and shares them with the product manager for review. After the product manager approves, the release manager opens a PR against [wso2/docs-integrator](https://github.com/wso2/docs-integrator) and ensures it is merged and published before the GA build ships. If any fixes affect documented behavior, the corresponding documentation updates _should_ be included in the same PR.

### Step 6: Team Verification

Each product team member _should_ install the pre-release build, verify their changes, and check off their PRs in the checklist issue. If a blocker-level issue is found, the fix author merges the fix to `<major>.<minor>.x`; the release manager triggers a new pre-release build and adds a new RC section to the checklist issue. Repeat until no blockers remain.

### Step 7: Trigger the Release Workflow

The release manager triggers `release-vsix.yml` targeting `<major>.<minor>.x`, then follows Steps 8–9 from the Feature Release Process.

### Step 8: Post-Release Steps

1. **Verify the tag** — confirm `v<major>.<minor>.<patch>` was created on the release commit.
2. **Confirm the GitHub Release** — verify artifacts and release notes are published.
3. **Confirm documentation is live** — verify the release notes PR is merged to [wso2/docs-integrator](https://github.com/wso2/docs-integrator) and published.
4. **Merge to `main`** — merge the maintenance branch into `main` so the fixes are included in the next feature release.
5. **Update the milestones** — close the release milestone and create the milestone for the next immediate patch release (e.g. `5.0.2`).
6. **Communicate** — notify the team; for security fixes, describe the fix without exposing exploit details.

## Hotfix Release Process

A hotfix release is for critical issues (security vulnerabilities, data loss, a broken release) that cannot wait for the normal patch cycle. Unlike a patch release, a hotfix isolates the critical fix on top of exactly what users are running. Use it when the maintenance branch contains other unreleased changes that cannot be validated in time.

### Step 1: Create the Hotfix Branch

Create `hotfix/<description>` from the latest stable release tag (e.g. `hotfix/critical-auth-bypass` from `v5.0.2`).

### Step 2: Apply and Verify the Fix

Open a PR against the hotfix branch with the minimal fix. All PR pipeline gates _must_ pass — the urgency does not waive any gate.

### Step 3: Create the Pre-Release Build

The release manager triggers a pre-release build from the hotfix branch and shares the artifact with the fix author, asking them to verify the issue is resolved by testing locally. The release manager _should_ also draft brief release notes describing the issue and the fix, and share them with the product manager for review.

If the fix is ineffective or introduces a regression, the fix author applies a follow-up fix to the hotfix branch and the release manager triggers a new pre-release build (repeat Step 3). Once the fix author confirms no blockers, the release manager proceeds to Step 4.

### Step 4: Trigger the Release Workflow

The release manager triggers `release-vsix.yml` targeting the hotfix branch, then runs `publish-vsix.yml` once the build is verified.

The hotfix takes the next patch version (e.g. `5.0.3` on top of `v5.0.2`); after the merge-back, the maintenance branch continues from that version.

### Step 5: Verify the Release

Confirm the `v<major>.<minor>.<patch>` tag was created on the release commit, and the artifacts and release notes are published.

### Step 6: Merge Back

1. Merge the hotfix branch into the active `<major>.<minor>.x` maintenance branch.
2. Merge the fix into `main` if it applies to the current development version.
3. Delete the hotfix branch.

### Step 7: Communicate

Notify the team and affected users. For security fixes, publish an advisory without exposing exploit details. The release manager _should_ ensure the release notes are merged to [wso2/docs-integrator](https://github.com/wso2/docs-integrator) and published.

## Pending Items

The following items represent gaps between this proposal and the current state of the repos.

- **No automated Nightly publish.** The continuous nightly publish described in Release Schedule does not yet exist. Merges to `main` do not trigger a publish in any repo. The pipeline needs to be built before the schedule can be followed.
