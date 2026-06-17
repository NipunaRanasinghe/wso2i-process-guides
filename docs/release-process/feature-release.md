# Feature Release Process

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/10 \
_Updated_: 2026/06/17

> For release types, schedule, and release order, see the [Release Process overview](README.md).

### Step 1: Create the Release Milestone

The release cycle starts when the release manager creates a public release milestone in the `product-integrator` repo (e.g. [WSO2 Integrator 5.1.0](https://github.com/wso2/product-integrator/milestone/7)), if one does not already exist. All features and fixes planned for the release must be tracked against this milestone.

### Step 2: Initiate the Release Thread

The release manager initiates a dedicated thread in the integration team chat with the release version as the topic. The thread must include the planned scope of the release and the target code freeze date. This thread is the primary communication channel between all release stakeholders. Product team members should use it to raise release blockers, flag concerns, and track decisions throughout the release process.

### Step 3: Verify the Release Readiness

Before triggering the release workflow, the release manager should verify:

- [ ] All planned features for this release are completed
- [ ] All issues added in the release milestone are either closed or moved to a future milestone
- [ ] All active patch branches are merged to the main branch (All bug fixes and security patches are merged to the active patch branch. See the [Branching-strategy](../branching-strategy.md#patch-branch-majorminorx) documentation for more details)
- [ ] No critical issues or regressions are found in the latest nightly build and the build is stable enough for release

Once the checklist is satisfied, the release manager must announce a code freeze on `main` and create the `<major>.<minor>.x` patch branch from the current `main` HEAD (e.g., for a `5.1.0` release, create `5.1.x`). From this point, only blocker-level issues and security fixes are permitted. Contributors must target `5.1.x`, not `main`, for any such fixes.

### Step 4: Pre-Release Builds

Feature releases progress through three pre-release stages before the GA build. All pre-release builds target the `<major>.<minor>.x` patch branch.

**Alpha:** the release manager triggers the first pre-release build from the patch branch and shares it with the immediate team for early testing. The focus is catching integration problems; known issues are expected at this stage. Blockers are fixed directly on the patch branch.

**Beta:** once alpha blockers are resolved, the release manager triggers a beta build from the patch branch. This is the build for broader internal testing. The release checklist (Step 5) and documentation (Step 6) should be prepared in parallel during the beta phase.

**RC:** once beta testing is complete and no blockers remain, the release manager triggers RC1 from the patch branch. This is the final candidate submitted for team verification (Step 7). If a blocker is found during RC verification, it is fixed on the patch branch and a new RC (RC2, RC3, …) is triggered. Only when the RC is verified clean does the release manager proceed to the GA build.

> See the [CI/CD Pipelines](../cicd-pipelines.md#release-pipelines) guide for the detailed steps of the pre-release pipeline.

### Step 5: Create and Share the Release Checklist

The release manager creates a GitHub issue in `product-integrator` repo, by listing all PRs included in the release as checkboxes (grouped by component and product team member), and shares the link with the team.

> Refer to the [release checklist example](https://github.com/wso2/product-ballerina-integrator/issues/2534) for the expected format.

### Step 6: Prepare Release Documentation

The release manager initiates two documentation efforts alongside the team verification. Both must be completed before the GA build ships.

**Release notes:** The release manager drafts release notes covering the changes in this release and shares them with the product manager for review. After the product manager approves, the release manager opens a PR against [wso2/docs-integrator](https://github.com/wso2/docs-integrator). The PR must be merged and the release notes published by the time the GA build goes live.

**Documentation updates:** The release manager coordinates with the product team members responsible for each change to update the [Integrator documentation website](https://github.com/wso2/docs-integrator). All documentation PRs must be reviewed, approved, and merged to the docs repo before the GA release is published.

### Step 7: Team Verification

Each product team member should install the pre-release build, verify their changes, and check off their PRs in the checklist issue.

If a blocker-level issue is found during verification:

1. The fix author merges the fix to `<major>.<minor>.x`.
2. The release manager triggers a new RC build from the patch branch (RC2, RC3, …).
3. The release manager adds a new RC section to the existing checklist issue (e.g. **RC2**) listing the additional PRs, and re-shares the link.
4. The team should verify the new build and check off the new items.

Repeat until no blocker-level issues remain. Non-blocking issues may be deferred to a future release with the release manager's approval. Once all checklist items are verified and no blockers remain, the release manager proceeds to Step 8.

### Step 8: Trigger the Release

The release manager triggers the plugin build workflow for each of the four plugin repos (`ballerina-tooling`, `mi-tooling`, `si-tooling`, and the WSO2 Integrator extension in `product-integrator`), reviews the draft GitHub Releases, then publishes all four to the VS Code Marketplace and OpenVSX Registry. Once all plugins are published, the release manager triggers the IDE release workflow in `product-integrator` to produce and publish the final IDE installers.

> See the [CI/CD Pipelines](../cicd-pipelines.md#release-pipelines) guide for the detailed steps of the release pipeline.

### Step 9: Post-Release Steps

After the GA artifacts are published:

1. **Verify the tag:** confirm `v<major>.<minor>.<patch>` was created on the release commit.
2. **Confirm the maintenance branch:** `<major>.<minor>.x` was created at Step 3; verify it points to the GA release commit and retire the previous maintenance branch (no further releases are created from it; the branch is kept for history).
3. **Bump `main`:** open a PR on `main` incrementing to the next minor dev version (e.g. `1.3.0-dev`).
4. **Confirm the GitHub Release:** verify the `product-integrator` bundle is published to [GitHub Releases](https://github.com/wso2/product-integrator/releases) with release notes.
5. **Confirm documentation is live:** verify all documentation update PRs and the release notes PR are merged to [wso2/docs-integrator](https://github.com/wso2/docs-integrator) and published on the website.
6. **Update the milestones:** close the release milestone and create the milestone for the next immediate patch release (e.g. `5.1.1`).
7. **Communicate:** notify the team and any affected stakeholders.
