# Repo State Snapshot

_Captured_: 2026/06/12 \
_Source_: GitHub Actions workflow files read directly from each repo

This document records the observed state of all five WSO2 Integrator tooling repos as of the capture date. It is intended as a reference baseline for drift checks against the proposal docs. Update this document whenever a significant pipeline change is made to any repo.

## Table of Contents

- [wso2/product-integrator](#wsо2product-integrator)
- [wso2/ballerina-vscode (ballerina-tooling)](#wso2ballerina-vscode-ballerina-tooling)
- [wso2/mi-vscode (mi-tooling)](#wso2mi-vscode-mi-tooling)
- [wso2/vscode-extensions](#wso2vscode-extensions)
- [siddhi-io/siddhi-plugin-vscode (si-tooling)](#siddhi-iosiddhi-plugin-vscode-si-tooling)
- [Cross-Cutting Observations](#cross-cutting-observations)

---

## wso2/product-integrator

**Workflow files:** `pr-ci.yml`, `compile.yml`, `build-and-release.yml`, `daily-pre-release-builds.yml`, `smoke-test.yml`, `resign-and-rehash.yml`

### PR Pipeline (`pr-ci.yml`)

- **Trigger:** `pull_request` on `main`, `5.0.x`; also `workflow_dispatch`
- **Concurrency:** group `pr-ci-<PR number or ref>`, cancel-in-progress: true
- **Jobs:** Calls `compile.yml` with `runOnAWS: false`, `build_packed_installers: false`, `ballerina_extension_source: github`, `mi_extension_source: github`
- **No Trivy scan.** No unit, integration, or E2E tests.

### Daily Build (`daily-pre-release-builds.yml`)

- **Trigger:** cron `30 6 * * *` (06:30 UTC / 12:00 IST) + `workflow_dispatch`
- **Jobs:** Dispatches `build-and-release.yml` for two targets:
  - `main` branch — version 5.1.0 pre-release, `delivery_mode: archive`
  - `5.0.x` branch — version 5.0.0 pre-release, `delivery_mode: archive`
- Does **not** publish to GitHub Releases or any marketplace.

### Compile (`compile.yml`, reusable)

This is the core build workflow, called by both `pr-ci.yml` and `build-and-release.yml`. It does **not** simply assemble pre-built extensions — it compiles a VS Code fork from source.

**Jobs:**
1. `resolve-versions` — reads `ci/build/component-versions.properties` and outputs: `ballerina_version`, `ballerina_extension_version`, `mi_extension_version`, `icp_version`, `integrator_version`, `ballerina_jre_version`
2. `validate-Inputs` — runs only when `build_packed_installers: true`; validates that required versions are set and correctly formatted (no leading `v`)
3. `compile` — runs in `lib/vscode` working directory (the VS Code fork); key steps:
   - Checkout with `submodules: recursive`
   - Sets up Node.js (from `.nvmrc`), Rush (via `gigara/setup-rush`), and Rust toolchain
   - Caches: built-in extensions, Rust toolchain, CLI compilation, Linux sysroots, apt packages
   - Downloads Ballerina and/or MI VSIX from GitHub Releases if `*_extension_source: github`
   - Runs `update-product.sh` to apply WSO2 branding/`product.json` patches
   - Builds the WI extension (in `wi/` directory) with Rush
   - Compiles VS Code TypeScript source (`npm run extensions-ci`, `npm run core-ci-pr`)
   - Builds Linux x64 client via `npm run gulp vscode-linux-x64-min-ci`
   - **If `build_packed_installers: true`:** downloads Ballerina runtime, ICP (Integration Control Plane), and a custom JRE; builds DEB, RPM, and TAR packed installers; signs DEB and RPM with `cosign` (keyless OIDC); uploads to AWS S3
   - **If `build_packed_installers: false`:** compiles Rust CLI; builds DEB and RPM using VS Code gulp tasks (Integrator-only, no bundled runtime)
   - Artifact uploads skipped on `pull_request` events

**Bundled components in packed installers:**
- Ballerina runtime (downloaded from `ballerina-platform/ballerina-distribution`)
- ICP — Integration Control Plane (downloaded from `wso2/integration-control-plane`)
- Custom JRE (downloaded from `ballerina-platform/ballerina-custom-jre`)
- Ballerina VS Code Extension (from marketplace or GitHub Releases)
- MI VS Code Extension (from marketplace or GitHub Releases)

### Release (`build-and-release.yml`)

- **Trigger:** `workflow_dispatch` only
- **Key inputs:** `build_linux`, `build_macos`, `build_windows`, `build_packed_installers`, `ballerina_extension_source` (marketplace/github), `mi_extension_source` (marketplace/github), `delivery_mode` (archive/github-release)
- Reads component versions from `ci/build/component-versions.properties`
- Calls `compile.yml` for the Linux build; macOS and Windows have dedicated jobs
- macOS build runs on `macos-latest`, matrix: `arch: [arm64]`
- Smoke tests called as a separate reusable workflow (`smoke-test.yml`) when packed installers are built

### Smoke Tests (`smoke-test.yml`)

- **Trigger:** `workflow_call` + `workflow_dispatch`
- Runs on `ubuntu-latest` and `windows-latest`, timeout 25 min
- Matrix: `hello-world-service`, `icp` (ICP is `continue-on-error: true`)
- Uses `wso2ipw@0.1.5` npm tool; ICP tests use Playwright + Chrome

### Resign and Rehash (`resign-and-rehash.yml`)

- **Trigger:** `workflow_dispatch`
- Signs DMG and MSI artifacts with `cosign` (keyless OIDC), generates SHA-256 hashes, uploads `.pem`/`.sig`/`.sha256` back to the release

### Runners

- PR and orchestration jobs: `ubuntu-latest`
- Build jobs: `codebuild-wso2_product-integrator-<run_id>-<attempt>` (AWS CodeBuild) or `ubuntu-latest` depending on input

### Branching

- Active branches: `main` (5.1.0 dev), `5.0.x` (maintenance)

---

## wso2/ballerina-vscode (ballerina-tooling)

**Workflow files:** `pull-request.yml`, `build.yml`, `daily-build.yml`, `release-vsix.yml`, `publish-vsix.yml`, `ls-publish-release.yml`, `sync-main-with-releases.yml`, `cache-cleanup.yml`

### PR Pipeline (`pull-request.yml`)

- **Trigger:** `pull_request` (opened, reopened, synchronize, ready_for_review) + `workflow_dispatch`
- **Concurrency:** group `${{ github.ref }}`, cancel-in-progress: true
- **Draft PRs:** skipped
- **Jobs:**
  - `changes` — uses `dorny/paths-filter` to detect changes in `packages/`, `submodules/wso2-vscode-extensions/workspaces/common-libs/`, `common/`, `rush.json`, `rush-config.json`, `.github/workflows/`
  - `build` — calls `build.yml`; opt-in AWS runner via label `Runner/AWS`; opt-in Ballerina E2E via label `Checks/Run Ballerina UI Tests` or `base_ref == 'stable/ballerina'`

### Build (`build.yml`, reusable)

- **Trigger:** `workflow_call` only
- **Jobs:**
  - `Build_Stage` — checkout with `submodules: recursive`; calls `./.github/actions/build`; runs **Trivy** filesystem scan (exit-code 1, skips `common/temp`, `submodules`, LS test resources, `--ignore-unfixed`); timeout 45 min
  - `ExtTest_Ballerina` — **disabled** (`if: false`)
  - `ExtTest_Ballerina_Diagrams` — diagram snapshot tests (BI diagram, component diagram, type diagram, sequence diagram); triggers on `runTests`, release build, diff, or `base_ref == 'stable/ballerina'`
  - `BalE2ETest` — Playwright E2E, matrix: 4 groups; triggers on `runBalE2ETests` or release+ballerina; re-runs on failure with `--last-failed`
- **env:** `ballerina_version: 2201.13.2`

### Daily Build (`daily-build.yml`)

- **Trigger:** cron `30 1 * * *` (01:30 UTC / 07:00 IST) + `workflow_dispatch`
- **Jobs:** Resolves target branches (main + latest `1.x.x` release branch); Gradle LS pack + test on Ubuntu (JDK 21); Gradle build on Windows; Trivy SBOM scan (generates CycloneDX SBOM via `cyclonedxBom`); calls `build.yml` with `ballerina: true`, `runTests: true`, `runBalE2ETests: true`; Google Chat notifications on success/failure

### Release — Extension (`release-vsix.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `isPreRelease`, `lsSource` (local/download), `ballerinaLsTag`, `version` (patch/minor/major/N/A)
- **Jobs:**
  - `Build` — calls `build.yml` with `runOnAWS: true`, `isReleaseBuild: true`, `enableCache: false`, `ballerina: true`
  - `Release` — AWS CodeBuild runner; restores build artifact; gets VSIX version; posts release thread to Google Chat; calls `./.github/actions/release` (creates **draft** GitHub Release); calls `./.github/actions/pr` (creates sync PR); notifies

### Publish — Extension (`publish-vsix.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `isPreRelease`, `vscode` (bool), `openVSX` (bool), `notify` (bool), `workflowRunId`
- **Jobs:**
  - `publish` — AWS CodeBuild runner; downloads VSIX from prior run (`workflowRunId`); publishes to VS Code Marketplace (`vsce`) and/or OpenVSX (`ovsx`); if stable: promotes draft GitHub Release to published; triggers `repository_dispatch` on external cloud editor builder repo; Google Chat notification

### Release — Language Server (`ls-publish-release.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `is_prerelease`, `skip_tests`
- **Jobs:**
  - Generates CycloneDX SBOM; runs **Trivy** SBOM scan (CRITICAL/HIGH, exit-code 1); resolves version (RC versioning for pre-release); creates/checks out `release-<BASE_VERSION>` branch; runs `gradlew release`; builds; publishes to GitHub Packages; creates GitHub Release via `gh release create --generate-notes`; if stable: creates post-release PR to `main`

### Sync Main (`sync-main-with-releases.yml`)

- **Trigger:** `pull_request` closed on `stable/ballerina**` branches (merged only)
- Creates temp branch from release branch, opens PR to `main`, notifies Google Chat

### Branching

- Active branches: `main`, `stable/ballerina` (release branch pattern — **not** `<major>.<minor>.x`)
- Submodule: `submodules/wso2-vscode-extensions` (vscode-extensions repo)

---

## wso2/mi-vscode (mi-tooling)

**Workflow files:** `test-pr.yml`, `build.yml`, `daily-build.yml`, `release-vsix.yml`, `publish-vsix.yml`, `sync-main-with-releases.yml`, `cache-cleanup.yml`

### PR Pipeline (`test-pr.yml`)

- **Trigger:** `pull_request` (opened, reopened, synchronize, ready_for_review) + `workflow_dispatch`
- **Concurrency:** group `${{ github.ref }}`, cancel-in-progress: true
- **Draft PRs:** skipped
- **Jobs:** Calls `build.yml`; opt-in AWS via `Runner/AWS` label; opt-in E2E via `Checks/Enable UI Tests`; opt-in MI E2E via `Checks/Run MI UI Tests` or `base_ref == 'stable/mi'`

### Build (`build.yml`, reusable)

- **Trigger:** `workflow_call` only
- **Jobs:**
  - `Build_Stage` — checkout `fetch-depth: 2`; diff analysis (`packages/`, `submodules/vscode-extensions`, `common/`, `.github/`, `package.json`, `pnpm-workspace.yaml`, `rush.json`); calls `./.github/actions/build`; runs **Trivy** filesystem scan (exit-code 1, skips `common/temp`, `packages/mi-language-server`)
  - `ExtTest_MI` — MI diagram tests via `xvfb-run pnpm run test` in `packages/mi-diagram`; triggers on `runTests`, release+mi, diff, or `base_ref == 'stable/mi'`
  - `UITest` — Playwright E2E on Linux (xvfb + ffmpeg) and Windows; 4 groups; re-run on failure; uploads test results, recordings, logs

### Daily Build (`daily-build.yml`)

- **Trigger:** cron `30 13 * * *` (13:30 UTC / 19:00 IST)
- **Jobs:** Calls `build.yml` with `mi: true`, `runTests: true`, `runMIE2ETests: true`; separate Google Chat notifications for Editor team and MI team

### Release — Extension (`release-vsix.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `isPreRelease`, `version` (patch/minor/major/N/A)
- Same two-step pattern as `ballerina-vscode`: builds VSIX → creates **draft** GitHub Release → sync PR

### Publish — Extension (`publish-vsix.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `isPreRelease`, `vscode`, `openVSX`, `workflowRunId`
- Downloads VSIX; publishes to VS Code Marketplace / OpenVSX; promotes draft release; sends marketplace release notification + MI team chat announcement

### Sync Main (`sync-main-with-releases.yml`)

- **Trigger:** `pull_request` closed on `stable/**` branches (merged only)

### Branching

- Active branches: `main`, `stable/mi` (release branch pattern — **not** `<major>.<minor>.x`)
- Submodule: `submodules/vscode-extensions`

---

## wso2/vscode-extensions

**Workflow files:** `test-pr.yml`, `build.yml`, `daily-build.yml`, `release-vsix.yml`, `release-packages.yml`, `publish-vsix.yml`, `save-cache.yml`, `sync-main-with-releases.yml`, `cache-cleanup.yml`

### PR Pipeline (`test-pr.yml`)

- **Trigger:** `pull_request` (opened, reopened, synchronize, ready_for_review) + `workflow_dispatch`
- **Concurrency:** group `${{ github.ref }}`, cancel-in-progress: true
- **Draft PRs:** skipped
- **Jobs:** Calls `build.yml`; opt-in AWS via `Runner/AWS` label; opt-in E2E via `Checks/Enable UI Tests`

### Build (`build.yml`, reusable)

- **Trigger:** `workflow_call` only
- **Extensions covered:** `wso2-platform`, `choreo`, `apk`, `hurl-client`, plus `common-libs` (shared UI libraries)
- **Jobs:**
  - `Build_Stage` — diff analysis per workspace; calls `./.github/actions/build`; runs **Trivy** filesystem scan (exit-code 1, skips `common/temp`)
  - `UITest` — Playwright E2E (matrix-driven; currently no entries produce real runs)

### Daily Build (`daily-build.yml`)

- **Trigger:** cron `30 13 * * *` (13:30 UTC / 19:00 IST)
- **Jobs:** Calls `build.yml` (all extensions); Google Chat notification via `TOOLING_TEAM_CHAT_API`

### Release — Extension (`release-vsix.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `isPreRelease`, `wso2-platform`, `choreo`, `apk`, `hurl-client` (booleans), `version`
- Builds selected extensions; creates draft GitHub Releases in respective repos (e.g. `wso2/choreo-vscode` for Choreo); opens sync PRs

### Release — Packages (`release-packages.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `isPreRelease`, `package` (choice: `common-libs/ui-toolkit` or `common-libs/font-wso2-vscode`), `version`
- Publishes shared UI libraries packages to GitHub Packages NPM registry (`npm.pkg.github.com`) via `pnpm publish`; creates version bump PR

### Publish — Extension (`publish-vsix.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `extension` (choice: choreo/wso2-platform/apk/hurl-client), `isPreRelease`, `vscode`, `openVSX`, `notify`, `workflowRunId`
- Downloads VSIX; publishes to VS Code Marketplace / OpenVSX; promotes draft release in the appropriate repo

### Cache Warming (`save-cache.yml`)

- **Trigger:** `push` to `dev` branch
- Builds the Rush monorepo to warm the build cache

### Branching

- Active branch: `main`

---

## siddhi-io/siddhi-plugin-vscode (si-tooling)

**Workflow files:** `test-pr.yml`, `build.yml`, `release.yml`, `publish-marketplace-vsix.yml`, `cache-cleanup.yml`

### PR Pipeline (`test-pr.yml`)

- **Trigger:** `pull_request` (opened, reopened, synchronize, ready_for_review) + `workflow_dispatch`
- **Concurrency:** group `${{ github.ref }}`, cancel-in-progress: true
- **Draft PRs:** skipped
- **Jobs:** Calls `build.yml` with defaults only (no extra inputs)
- **No Trivy scan.** No tests beyond build + package.

### Build (`build.yml`, reusable)

- **Trigger:** `workflow_call` only
- **Jobs:**
  - `Build_Stage` — `ubuntu-latest`, timeout 45 min; Node 22.x; `install-run-rush.js build`; packages VSIX via `npm run package`; uploads artifact
- **No submodule checkout.** Does not consume `vscode-extensions` shared UI libraries.
- **No Trivy scan.**

### No daily build workflow.

### Release (`release.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `prerelease` (bool), `prereleaseVersion` (optional string override, e.g. `"0.99"`)
- **Versioning:**
  - Stable: reads version from `package.json`
  - Pre-release: uses `major.odd_minor.epoch_minutes` scheme (not SemVer `-nightly` suffix)
- **Jobs:**
  - Node 22.x + Rush; resolves version; if pre-release: overwrites `package.json` version before build; builds; uploads artifact; creates GitHub Release with `gh release create --generate-notes`
  - If stable: bumps minor by +2 (to next even minor), commits, pushes, opens post-release PR
- Uses `WSO2_INTEGRATION_BOT_TOKEN` / `WSO2_INTEGRATION_BOT_USERNAME` / `WSO2_INTEGRATION_BOT_EMAIL`

### Publish (`publish-marketplace-vsix.yml`)

- **Trigger:** `workflow_dispatch`
- **Inputs:** `tag` (release tag, e.g. `v0.1.0`), `isPreRelease`, `vscode`, `openVSX`
- Downloads VSIX from GitHub Release via `gh release download`; publishes via `vsce`; OpenVSX skipped for pre-releases
- **No AWS runner.** Runs on `ubuntu-latest` only.

### Branching

- Active branch: `main`

---

## Cross-Cutting Observations

### Runners

| Repo | PR jobs | Release/build jobs |
|---|---|---|
| `ballerina-vscode` | `ubuntu-latest` (AWS opt-in via label) | AWS CodeBuild (`codebuild-*`) |
| `mi-vscode` | `ubuntu-latest` (AWS opt-in via label) | AWS CodeBuild |
| `vscode-extensions` | `ubuntu-latest` (AWS opt-in via label) | AWS CodeBuild |
| `product-integrator` | `ubuntu-latest` | `codebuild-wso2_product-integrator-*` or `ubuntu-latest` |
| `siddhi-plugin-vscode` | `ubuntu-latest` | `ubuntu-latest` (no AWS runner) |

### Approval Gates

No repo uses a GitHub Actions Environment with required reviewers. The closest mechanism is the two-step manual `workflow_dispatch` pattern used by `ballerina-vscode`, `mi-vscode`, and `vscode-extensions`: a release manager runs `release-vsix.yml` to build and create a draft release, then separately runs `publish-vsix.yml` to publish. The person running the publish step is the implicit approver.

### Quality Gates by Repo

| Repo | Trivy (PR) | Trivy (daily SBOM) | SonarQube | Secret Scanning |
|---|---|---|---|---|
| `ballerina-vscode` | Yes (filesystem) | Yes (CycloneDX SBOM) | Not configured | Repo-level setting |
| `mi-vscode` | Yes (filesystem) | No | Not configured | Repo-level setting |
| `vscode-extensions` | Yes (filesystem) | No | Not configured | Repo-level setting |
| `product-integrator` | No | No | Not configured | Repo-level setting |
| `siddhi-plugin-vscode` | No | No | Not configured | Repo-level setting |

### Shared UI Libraries Consumption

| Repo | Consumes `vscode-extensions`? | Method |
|---|---|---|
| `ballerina-vscode` | Yes | Git submodule (`submodules/wso2-vscode-extensions`), built from source |
| `mi-vscode` | Yes | Git submodule (`submodules/vscode-extensions`), built from source |
| `vscode-extensions` | N/A — this is the libraries repo | — |
| `product-integrator` | Unknown — not evident from workflow files | — |
| `siddhi-plugin-vscode` | No | No submodule; does not use shared libraries |

### Daily Build Schedule Summary

| Repo | UTC | IST |
|---|---|---|
| `ballerina-vscode` | 01:30 | 07:00 |
| `mi-vscode` | 13:30 | 19:00 |
| `vscode-extensions` | 13:30 | 19:00 |
| `product-integrator` | 06:30 | 12:00 |
| `siddhi-plugin-vscode` | — | No daily build |
