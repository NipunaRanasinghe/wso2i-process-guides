# Versioning Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/14

This document defines the versioning scheme applied across all WSO2 Integrator repos. The common rule (SemVer) is stated first, followed by how each versioned unit applies it (shared UI library, language server, VS Code extension, and product distribution), and finally the rules that govern version flow _between_ repos.

## SemVer

All repos _must_ follow [Semantic Versioning (SemVer)](https://semver.org/).

> Given a version number MAJOR.MINOR.PATCH, increment the:
>
> - MAJOR version when you make incompatible API changes
> - MINOR version when you add functionality in a backward compatible manner
> - PATCH version when you make backward compatible bug fixes

Version bumps in manifests (`package.json`, `pom.xml`) are applied as part of the release workflow, via automated version-increment scripts invoked by the release pipeline. How each versioned unit interprets and applies SemVer is described below.

## Shared UI Library Versioning

The shared UI libraries **have no independent version line and are never released as a package**. Consumers pin them as a **git submodule commit** and build them from source. The "version" of the libraries a product consumes is therefore the submodule commit it points at — not a SemVer tag.

Because there is no published artifact, SemVer MAJOR/MINOR/PATCH semantics do not apply to the libraries directly; the impact of a library change is realized in the SemVer bump of the *consuming* product extension when it moves its submodule pointer forward.

## Language Server Versioning

Language servers are versioned and released together with their parent extension — there is no independent language server release. Each product version includes a specific language server build, so consumers always receive an extension and language server combination that was tested together.

> **Note:** If an external consumer of a language server emerges in the future, the repo structure supports promoting it to an independent SemVer line without structural change.

## VS Code Extension Versioning

Each product tooling repo carries its own SemVer line, applied at the **extension** level. The extension is the unit published to the VS Code Marketplace, and it bundles a matched [language server](#language-server-versioning) build. Extensions publish on two Marketplace channels, each with its own versioning rule.

**Stable:** A plain SemVer `major.minor.patch` version (e.g. `1.2.0`), published to the Marketplace **stable** channel via `vsce publish`. Produced by the Stable/GA pipeline (see [CI/CD Pipelines](cicd-pipelines.md#release-pipelines)).

**Pre-release:** Nightly builds published to the Marketplace **pre-release** channel via `vsce publish --pre-release`. The Marketplace does **not** support SemVer pre-release suffixes — `1.2.0-nightly.20260609` is rejected; every published version _must_ be `major.minor.patch`. Pre-release builds therefore encode the build timestamp in the **patch** segment so each build sorts strictly above the previous one — for example `1.2.2606141230` (`major.minor.<YYMMDDHHmm>`).

> **Note:** VS Code also offers an [even-minor/odd-minor convention](https://code.visualstudio.com/api/working-with-extensions/publishing-extension#prerelease-extensions) — stable on even minors, pre-release on odd — to keep the two channels from sharing a version number. It is optional: the `--pre-release` flag already separates the channels, so repos may adopt it or not.

## Product Distribution Versioning

The `product-integrator` artifacts (the WSO2 Integrator VS Code Extension and the WSO2 Integrator IDE) are versioned with the **WSO2 Integrator product version** — `5.0.0` for the first consolidated release — **not** a repo-level SemVer line.

The product version is managed at the WSO2 Integrator product level and reflects the bundled set as a whole: component versions evolve independently, and users upgrade WSO2 Integrator as a single product. In the repo, the product version is pinned alongside the component versions in [`ci/build/component-versions.properties`](https://github.com/wso2/product-integrator/blob/main/ci/build/component-versions.properties).

## Cross-Repo Version Propagation

Cross-repo dependencies are pinned to explicit references: the shared UI libraries as a git submodule commit, and product extensions as version pins (see [Cross-Repo Coordination](cicd-pipelines.md#cross-repo-coordination)). A change in a dependency therefore **never propagates automatically**: dependents adopt it through a PR that updates the pinned reference. Consumers are not required to be on the same libraries commit, and there is no convergence requirement before a WSO2 Integrator stable release.

### Shared UI Library → VS Code Extension (upstream-first)

Library changes _must_ follow an **upstream-first rule**: every change lands on `vscode-extensions` `main` first, and consumers adopt it by moving their submodule pointer forward — never by maintaining product-specific library commits that are not on `main`. This guarantees that a fix made for one product is available to all of them.

A breaking change on the shared libraries `main` affects every product repo — but only when each updates its submodule pointer, because the pinned commits keep existing builds working. The risk is an uncoordinated migration that leaves product repos pinned to old library commits indefinitely. Breaking changes _must_ follow this process:

1. The author opens an issue in `vscode-extensions` describing the change, the migration path, and the impacted repos, and tags the owners of each product repo.
2. The change is merged to `main`.
3. Each product repo owner opens a migration PR that updates the submodule pointer and applies the required code changes. All PR pipeline gates apply as usual.
4. `product-integrator` adopts the migrated product extensions through its normal dependency bump — no special handling is needed at the product distribution layer.

Product repos _should_ complete the migration before their next stable release. There is no fixed calendar deadline.

### VS Code Extension → Product Distribution

If `ballerina-tooling` releases a new version (e.g. `1.2.0`), nothing happens in `product-integrator` automatically — its bundling pipeline is not triggered by upstream releases. The `product-integrator` release manager bumps the pinned version as part of preparing the next WSO2 Integrator release, and the bundled extension set ships as a tested, matched combination.

## Pending Items

The following items represent gaps between this proposal and the current state of the repos.

- **Standardize pre-release timestamp granularity across repos:** All three product repos encode a build timestamp in the patch segment, but the granularity differs — `si-tooling` uses minute granularity, while `ballerina-tooling` and `mi-tooling` use `YYMMDDHH` hour granularity, which can collide when more than one build runs in the same hour. Minute granularity avoids collisions. Note: the VS Code Marketplace does not accept SemVer pre-release suffixes (e.g. `-nightly.x`), so the timestamp-in-patch approach _must_ be retained — a `-nightly` suffix is not a valid target.
- **Resolve the Ballerina Language Server independent release path:** `ballerina-tooling` contains a `ls-publish-release.yml` workflow that can publish the language server to GitHub Packages independently from the parent extension. This conflicts with the policy that language servers are versioned and released together with their parent extension. The workflow should either be removed or the policy updated to explicitly allow the independent release path.
