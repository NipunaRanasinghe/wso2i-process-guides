# Release Process

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/10 \
_Updated_: 2026/06/10

This document describes the human process for deciding when and how to cut a release. For the automated pipeline mechanics, see [CI/CD Pipeline Design](04-cicd-pipeline-design.md).

## Release Cadence

- **Nightly / Insider**: continuous — every merge to `main` produces a new Insider build automatically.
- **Stable / GA**: target every 4–6 weeks, or immediately for critical patch fixes. The release owner decides based on feature readiness and stability signals from the Insider channel.

## Release Ownership

Each stable release has a designated release owner — typically the team lead or a rotation. The release owner is responsible for triggering the workflow, shepherding the approval gate, and completing post-release steps.

## Pre-Release Checklist

Before triggering the stable release workflow:

- [ ] All planned features for this release are merged to `main`
- [ ] No critical open issues or regressions in the Insider channel
- [ ] Changelog updated for the release version
- [ ] Version bumped in all affected manifests (`package.json`, `pom.xml`) — see [Versioning Strategy](03-versioning-strategy.md)
- [ ] QA sign-off obtained on the latest Insider build

## Triggering a Stable Release

1. Go to the **Actions** tab of the target repo and select the stable release workflow (`release.yml`).
2. Click **Run workflow** and enter the target commit SHA or branch (`main` or `<major>.<minor>.x`).
3. The workflow builds and packages artifacts, then pauses at the `production` environment gate.
4. A required reviewer approves in the GitHub UI — the publish step proceeds.

## Post-Release Steps

After the GA artifacts are published:

1. **Verify the tag** — confirm `v<major>.<minor>.patch` was created on the release commit.
2. **Cut the maintenance branch** — create `<major>.<minor>.x` from the GA tag (skip if it already exists for a patch release).
3. **Bump `main`** — open a PR on `main` incrementing to the next minor dev version (e.g. `1.3.0-dev`).
4. **Confirm the GitHub Release** — verify the `product-integrator` bundle is published to [GitHub Releases](https://github.com/wso2/product-integrator/releases) with release notes.
5. **Communicate** — notify the team and update any public-facing changelogs or announcements.

## Patch Release Process

A patch release targets the `<major>.<minor>.x` maintenance branch, not `main`.

1. Confirm the fix qualifies — bug fixes and security patches only; no new features.
2. Open a PR against `<major>.<minor>.x` with the cherry-picked fix. All pipeline gates must pass.
3. Trigger the stable release workflow targeting `<major>.<minor>.x`.
4. After the patch ships, backport the fix to `main` if applicable.
