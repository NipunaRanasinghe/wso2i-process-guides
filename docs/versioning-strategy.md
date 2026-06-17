# Versioning Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/14

This document defines the versioning scheme applied across all WSO2 Integrator repos. The common rule (SemVer) is stated first, followed by how each versioned unit applies it: shared UI library, language server, VS Code extension, and product distribution.

## SemVer

All the versioned components should follow the 
[Semantic Versioning (SemVer)](https://semver.org/) standards.

> Given a version number MAJOR.MINOR.PATCH, increment the:
>
> - MAJOR version when you make incompatible API changes
> - MINOR version when you add functionality in a backward compatible manner
> - PATCH version when you make backward compatible bug fixes

How each versioned unit interprets and applies SemVer is described below.

## Shared UI Library Versioning

The shared UI libraries have no independent version line and are never released as a package. Consumers pin them as a git submodule commit and build them from source. The version a product consumes is the submodule commit it points at.

## Language Server Versioning

Language servers are versioned and released together with their parent extension. There is no independent language server release. Each product version includes a specific language server build, so the extension and its language server are tested together and ship as one unit.

> **Note:** If an external consumer of a language server emerges in the future, the repo structure supports promoting it to an independent SemVer line without structural change.

## VS Code Extension Versioning

Each product tooling repo carries its own SemVer line, applied at the **extension** level. The extension is the unit published to the VS Code Marketplace, and it bundles a matched [language server](#language-server-versioning) build. Extensions publish on two Marketplace channels, each with its own versioning rule.

**Stable:** A plain SemVer `major.minor.patch` version (e.g. `1.2.0`), published to the Marketplace stable channel via `vsce publish`. Produced by the Stable/GA pipeline (see [CI/CD Pipelines](cicd-pipelines.md#release-pipelines)).

**Pre-release:** Nightly builds published to the Marketplace pre-release channel via `vsce publish --pre-release`. The Marketplace does not support SemVer pre-release suffixes: `1.2.0-nightly.20260609` is rejected, and every published version must be `major.minor.patch`. Pre-release builds therefore encode the build timestamp in the patch segment so each build sorts strictly above the previous one, for example `1.2.2606141230` (`major.minor.<YYMMDDHHmm>`).

> **Note:** VS Code also offers an [even-minor/odd-minor convention](https://code.visualstudio.com/api/working-with-extensions/publishing-extension#prerelease-extensions) (stable on even minors, pre-release on odd) to keep the two channels from sharing a version number. It is optional: the `--pre-release` flag already separates the channels, so repos may adopt it or not.

## Product Distribution Versioning

The `product-integrator` artifacts (the WSO2 Integrator VS Code Extension and the WSO2 Integrator IDE) are versioned with the **WSO2 Integrator product version** (`5.0.0` for the first consolidated release).

The product version is managed at the WSO2 Integrator product level. Component versions evolve independently, and users upgrade WSO2 Integrator as a single product. In the repo, the product version is pinned alongside the component versions in [`ci/build/component-versions.properties`](https://github.com/wso2/product-integrator/blob/main/ci/build/component-versions.properties).

## Pending Items

The following items are gaps between this proposal and the current state of the repos.

- **Standardize pre-release timestamp granularity across repos:** All three product repos encode a build timestamp in the patch segment, but the granularity differs: `si-tooling` uses minute granularity, while `ballerina-tooling` and `mi-tooling` use `YYMMDDHH` hour granularity, which can collide when more than one build runs in the same hour. Minute granularity avoids collisions. The VS Code Marketplace does not accept SemVer pre-release suffixes (e.g. `-nightly.x`), so the timestamp-in-patch approach must be retained. A `-nightly` suffix is not a valid target.
- **Resolve the Ballerina Language Server independent release path:** `ballerina-tooling` contains a `ls-publish-release.yml` workflow that can publish the language server to GitHub Packages independently from the parent extension. This conflicts with the policy that language servers are versioned and released together with their parent extension. The workflow should either be removed or the policy updated to explicitly allow the independent release path.
