# Patch Release Process

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/10 \
_Updated_: 2026/06/12

> For release types, schedule, and release order, see the [Release Process overview](README.md).

A patch release targets the `<major>.<minor>.x` maintenance branch, not `main`.

### Step 1: Confirm the Release Milestone

Confirm the dedicated milestone for this patch release exists in the `product-integrator` repo (e.g. [WSO2 Integrator 5.0.1](https://github.com/wso2/product-integrator/milestone/8)), and create it if it does not. All changes shipping in the release are tracked against this milestone.

### Step 2: Qualify the Fixes

Confirm every change qualifies — bug fixes and security patches only; no new features.

### Step 3: Create the Pre-Release Build

The release manager triggers a pre-release build from the target commit on `<major>.<minor>.x`. This publishes to the VS Code Marketplace pre-release channel and creates a pre-release tag on GitHub Releases, producing the build that the team _should_ install and verify.

### Step 4: Create and Share the Release Checklist

The release manager creates a GitHub issue in `product-integrator` titled `[Release Checklist] WSO2 Integrator <version>` with the `Type/Task` label, listing all PRs included in the release as GFM checkboxes grouped by repo and product team member, and shares the link with the team. Follow the same format as the [feature release checklist](feature-release.md#step-4-create-and-share-the-release-checklist).

### Step 5: Prepare Release Notes

The release manager drafts release notes for the patch and shares them with the product manager for review. After the product manager approves, the release manager opens a PR against [wso2/docs-integrator](https://github.com/wso2/docs-integrator) and ensures it is merged and published before the GA build ships. If any fixes affect documented behavior, the corresponding documentation updates _should_ be included in the same PR.

### Step 6: Team Verification

Each product team member _should_ install the pre-release build, verify their changes, and check off their PRs in the checklist issue. If a blocker-level issue is found, the fix author merges the fix to `<major>.<minor>.x`; the release manager triggers a new pre-release build and adds a new RC section to the checklist issue. Repeat until no blockers remain.

### Step 7: Trigger the Release Workflow

The release manager triggers `release-vsix.yml` targeting `<major>.<minor>.x`, then follows [Step 8](feature-release.md#step-8-publish-the-release) and [Step 9](feature-release.md#step-9-post-release-steps) from the Feature Release Process.

### Step 8: Post-Release Steps

1. **Verify the tag** — confirm `v<major>.<minor>.<patch>` was created on the release commit.
2. **Confirm the GitHub Release** — verify artifacts and release notes are published.
3. **Confirm documentation is live** — verify the release notes PR is merged to [wso2/docs-integrator](https://github.com/wso2/docs-integrator) and published.
4. **Merge to `main`** — merge the maintenance branch into `main` so the fixes are included in the next feature release.
5. **Update the milestones** — close the release milestone and create the milestone for the next immediate patch release (e.g. `5.0.2`).
6. **Communicate** — notify the team; for security fixes, describe the fix without exposing exploit details.
