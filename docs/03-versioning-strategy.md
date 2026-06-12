# Versioning Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/12

This document defines the versioning scheme applied across all WSO2 Integrator repos.

## SemVer

All repos _must_ follow [Semantic Versioning (SemVer)](https://semver.org/).

Excerpt from semver:

> Given a version number MAJOR.MINOR.PATCH, increment the:
>
> - MAJOR version when you make incompatible API changes
> - MINOR version when you add functionality in a backward compatible manner
> - PATCH version when you make backward compatible bug fixes

Version bumps in manifests (`package.json`, `pom.xml`) are updated as part of the release workflow, with automated version increment scripts invoked by the release pipeline.

## Cross-Repo Version Bumps

Cross-repo dependencies are pinned to explicit references — the shared UI toolkit as a git submodule commit, and product extensions as version pins (see [Cross-Repo Coordination](04-cicd-pipelines.md#cross-repo-coordination)). A change in a dependency therefore never propagates automatically — dependents adopt it through a PR that updates the pinned reference. Consumers are not required to be on the same toolkit commit, and there is no convergence requirement before a WSO2 Integrator stable release.

Toolkit changes _must_ follow an **upstream-first rule**: every change lands on `vscode-extensions` `main` first, and consumers adopt it by moving their submodule pointer forward — never by maintaining product-specific toolkit commits that are not on `main`. This guarantees that a fix made for one product is available to all of them.

### Toolkit Breaking Changes

A breaking change on toolkit `main` affects every product repo — but only when each updates its submodule pointer, because the pinned commits keep existing builds working. The risk is an uncoordinated migration that leaves product repos pinned to old toolkit commits indefinitely.

Breaking changes _must_ follow this process:

1. The author opens an issue in `vscode-extensions` describing the change, the migration path, and the impacted repos, and tags the owners of each product repo.
2. The change is merged to `main`.
3. Each product repo owner opens a migration PR that updates the submodule pointer and applies the required code changes. All PR pipeline gates apply as usual.
4. `product-integrator` adopts the migrated product extensions through its normal dependency bump — no special handling is needed at the product distribution layer.

Product repos _should_ complete the migration before their next stable release. There is no fixed calendar deadline.

### Adopting Product Extension Releases

If `ballerina-tooling` releases a new version (e.g. `1.5.0`), nothing happens in `product-integrator` automatically — its bundling pipeline is not triggered by upstream releases. The `product-integrator` release owner bumps the pinned version as part of preparing the next WSO2 Integrator release, and the bundled extension set ships as a tested, matched combination.

## Product Distribution Versioning

The `product-integrator` artifacts (the WSO2 Integrator VS Code Extension and the WSO2 Integrator IDE) are versioned with the **WSO2 Integrator product version** — `5.0.0` for the first consolidated release — rather than a repo-level SemVer line. The product version is managed at the WSO2 Integrator product level and reflects the bundled set as a whole: component versions evolve independently, and users upgrade WSO2 Integrator as a single product. In the repo, the product version is pinned alongside the component versions in [`ci/build/component-versions.properties`](https://github.com/wso2/product-integrator/blob/main/ci/build/component-versions.properties).

## Language Server Versioning

Language servers are versioned and released together with their parent extension — there is no independent language server release. Each product version (e.g. `ballerina-tooling 1.4.0`) includes a specific language server build; consumers always receive an extension and language server combination that was tested together.

> **Note:** If an external consumer of a language server emerges in the future, the repo structure supports promoting it to an independent release without structural change.

## Pending Items

The following items represent gaps between this proposal and the current state of the repos.

- **`si-tooling` uses a non-SemVer pre-release versioning scheme.** Pre-release builds currently use a `major.odd_minor.epoch_minutes` format. This needs to be migrated to a consistent SemVer pre-release approach (e.g. `1.2.0-nightly.20260609`) aligned with the other repos.
- **Ballerina Language Server has an independent release path.** `ballerina-tooling` contains a `ls-publish-release.yml` workflow that can publish the language server to GitHub Packages independently from the parent extension. This conflicts with the policy that language servers are versioned and released together with their parent extension. The workflow should either be removed or the policy updated to explicitly allow the independent release path.
