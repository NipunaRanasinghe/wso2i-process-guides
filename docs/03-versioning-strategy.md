# Versioning Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/11

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

Cross-repo dependencies are pinned to explicit versions (see [Cross-Repo Coordination](04-cicd-pipelines.md#cross-repo-coordination)). A new release of a dependency therefore never propagates automatically — dependents pick it up through a dependency-bump PR.

To illustrate the cases below, consider the following scenario:

- `vscode-extensions` has released its shared packages at `2.3.0`.
- `ballerina-tooling`, `mi-tooling`, and `si-tooling` pin the shared packages at `2.3.0` and are themselves at `1.4.0`.
- `product-integrator` pins each product extension at `1.4.0`.

### Case 1: Shared UI Toolkit Releases a Patch Version

If `vscode-extensions` releases `2.3.1`, product repos do not need to react. It is _recommended_ to pick up the patch in the next routine dependency bump, but because each extension bundles its own copy of the shared packages, staying on `2.3.0` causes no conflict.

### Case 2: Shared UI Toolkit Releases a Minor Version

If `vscode-extensions` releases `2.4.0`, each product repo that needs the new functionality _should_ open a dependency-bump PR updating the pin to `2.4.0` and include the bump in its next release. Product repos that do not need it may stay on `2.3.x` — unlike a shared runtime distribution, the bundled-dependency model tolerates product repos being on different shared UI toolkit minor versions. Product repos are not required to converge on a single toolkit minor version before a WSO2 Integrator stable release.

### Case 3: Shared UI Toolkit Releases a Major (Breaking) Version

If `vscode-extensions` releases `3.0.0`, every product repo is affected and a coordinated migration is required. Because dependencies are pinned, the release itself breaks nothing — product repos keep building against `2.x`. The risk is an uncoordinated migration that leaves product repos on diverging foundation versions indefinitely.

Breaking changes _must_ follow this process:

1. The author opens an issue in `vscode-extensions` describing the change, the migration path, and the impacted repos, and tags the owners of each product repo.
2. The change is merged and released as a new major version of the shared packages.
3. Each product repo owner opens a migration PR that bumps the pinned version and applies the required code changes. All PR pipeline gates apply as usual.
4. `product-integrator` picks up the migrated product extensions through its normal dependency bump — no special handling is needed at the product distribution layer.

Product repos _should_ complete the migration before their next stable release. There is no fixed calendar deadline.

### Case 4: A Product Extension Releases

If `ballerina-tooling` releases `1.5.0`, nothing happens in `product-integrator` automatically — its bundling pipeline does not follow upstream releases. The `product-integrator` release owner bumps the pinned version to `1.5.0` as part of preparing the next WSO2 Integrator release, and the bundled extension set ships as a tested, matched combination.

## Product Distribution Versioning

The `product-integrator` artifacts (the WSO2 Integrator VS Code Extension and the WSO2 Integrator IDE) carry the **WSO2 Integrator product version** — `5.0.0` for the first consolidated release — rather than a repo-level SemVer line. The product version is managed at the WSO2 Integrator product level and reflects the bundled set as a whole: component versions evolve independently, and users upgrade WSO2 Integrator as a single product. In the repo, the product version is pinned alongside the component versions in [`ci/build/component-versions.properties`](https://github.com/wso2/product-integrator/blob/main/ci/build/component-versions.properties).

## Language Server Versioning

Language servers are versioned and released together with their parent extension — there is no independent language server release. Each product version (e.g. `ballerina-tooling 1.4.0`) includes a specific language server build; consumers always receive a tested, matched pair.

> **Note:** If an external consumer of a language server emerges in the future, the repo structure supports promoting it to an independent release without structural change.
