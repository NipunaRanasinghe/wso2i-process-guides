# Feature Release Process

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/10 \
_Updated_: 2026/06/12

> For release types, schedule, and release order, see the [Release Process overview](README.md).

### Step 1: Create the Release Milestone

The release cycle starts when the release manager creates a public release milestone in the `product-integrator` repo (e.g. [WSO2 Integrator 5.1.0](https://github.com/wso2/product-integrator/milestone/7)), if one does not already exist. All features and fixes planned for the release must be tracked against this milestone.

### Step 2: Verify the Release Readiness

Before triggering the release workflow, the release manager should verify:

- [ ] All planned features for this release are completed
- [ ] All issues added in the release milestone are either closed or moved to a future milestone
- [ ] All active patch branches are merged to the main branch (All the bug fixes and security patches gets merged to the active patch branch. See the [Branching-strategy](../branching-strategy.md#patch-branch-majorminorx) documentation for more details)
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
