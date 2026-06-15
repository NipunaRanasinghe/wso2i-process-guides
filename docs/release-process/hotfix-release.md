# Hotfix Release Process

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/10 \
_Updated_: 2026/06/12

> For release types, schedule, and release order, see the [Release Process overview](README.md).

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
