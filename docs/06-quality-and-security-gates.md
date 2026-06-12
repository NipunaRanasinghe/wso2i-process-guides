# Quality & Security Gates

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/11

This document defines the quality and security gates integrated into the PR pipeline across all WSO2 Integrator repos, the tools used to implement them, and the blocking criteria for each gate.

| Requirement | Tool | Blocks Merge? | Notes |
|---|---|---|---|
| Code quality (coverage, duplication, complexity) | SonarQube Cloud | **Yes** | GitHub Actions step; posts results to the PR; free tier initially |
| Dependency vulnerability scanning | Trivy | **Yes** (`HIGH`/`CRITICAL`) | GitHub Actions step; already configured in all repos |
| Secret / token detection | GitHub Secret Scanning | **Yes** (push protection) | Enabled at repo level (Settings → Security); no pipeline step needed; free for public repos |

## SonarQube Cloud

SonarQube Cloud analyses each PR for code coverage, duplication, complexity, and maintainability issues, and posts the result directly to the PR as a status check.

- **Rationale:** Free for public repos, native GitHub PR decoration, and no self-hosted server to operate — the main alternatives (self-managed SonarQube, CodeQL-only) either add infrastructure cost or do not cover code quality metrics.
- **Blocking:** The quality gate status check _must_ pass before merge. Repos start with the default `Sonar way` quality gate; stricter per-repo quality gates (e.g. coverage thresholds) can be configured once a baseline exists.

## Trivy

Trivy scans dependency manifests (`npm` and Maven) for known vulnerabilities on every PR.

- **Rationale:** Already configured in all five repos, covers both ecosystems in use, and runs as a self-contained GitHub Actions step at no cost. 
- **Blocking:** A finding of severity `HIGH` or `CRITICAL` fails the PR pipeline. Lower severities are reported in the scan output but do not block; maintainers _should_ review them during routine dependency bumps.
- **Suppressions:** A `HIGH`/`CRITICAL` finding with no released fix may be suppressed (e.g. via a `.trivyignore` entry) only with explicit approval from the repo maintainers, case by case. Each suppression _must_ reference a tracking issue, and the entry _must_ be removed once a fixed version is available.

## GitHub Secret Scanning

GitHub Secret Scanning detects committed credentials (API tokens, keys) at the platform level. It is enabled per repo under **Settings → Security**, with **push protection** turned on, and requires no pipeline step.

- **Rationale:** Native to GitHub, no maintenance overhead, and free for public repos. Push protection rejects the push itself, which is preferable to detecting a secret after it has been committed to the repository history.
- **Blocking:** Push protection blocks any push containing a detected secret. Alerts on already-committed secrets _must_ be triaged by repo maintainers, and the affected credential _must_ be rotated — removing it from history alone is not sufficient.
