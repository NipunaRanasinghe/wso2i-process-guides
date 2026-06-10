# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This workspace contains the **CI/CD, release, versioning, and processes proposal** for the WSO2 Integrator tooling architecture revamp. There is no runnable code — the deliverable is a set of Markdown documents.

## Files

- `requirements.md` — The original brief: questions and answers from the team. The primary source of constraints and decisions.
- `WSO2 Integrator Architecture Revamp proposal.md` — Architecture context document. Describes existing issues, objectives, ownership split, and outstanding steps.
- `proposed-repo-structure.png` — Visual diagram of the proposed repository structure.

## Proposal Documents

The proposal is split into `docs/`, one document per topic. See `docs/README.md` for the full index.

| # | File | Topic |
|---|---|---|
| 1 | `docs/01-repository-structure.md` | Repo layer layout, dependency diagram, build-order constraints |
| 2 | `docs/02-branching-strategy.md` | `main` + `<major>.<minor>.x` maintenance branch model |
| 3 | `docs/03-versioning-strategy.md` | SemVer, language server versioning |
| 4 | `docs/04-cicd-pipelines.md` | PR and merge-to-main pipeline anatomy |
| 5 | `docs/05-testing-strategy.md` | Test types, pipeline stages, blocking gates |
| 6 | `docs/06-quality-and-security-gates.md` | GHAS + SonarQube Cloud |
| 7 | `docs/07-release-process.md` | Release cadence, ownership, checklist, post-release steps |

`docs/writing-style.md` — Style guide for all documents in this workspace, derived from the ballerina-platform/ballerina-library docs corpus. Apply it when creating or editing any doc.

## Repos

Use these to validate and verify any existing or future claims in the proposal docs.

| Repo (proposal name) | GitHub URL | Notes |
|---|---|---|
| `product-integrator` | https://github.com/wso2/product-integrator/ | IDE shell + WSO2 Integrator Extension |
| `ballerina-tooling` | https://github.com/wso2/ballerina-vscode/ | Pending rename to `ballerina-tooling` |
| `mi-tooling` | https://github.com/wso2/mi-vscode | Pending rename to `mi-tooling` |
| `vscode-extensions` | https://github.com/wso2/vscode-extensions | Shared foundation |
| `si-tooling` | _(URL not yet known)_ | Pending repo creation / rename |

## Key Decisions (resolved)

| Topic | Decision |
|---|---|
| CI/CD platform | GitHub Actions |
| Build tools | Gradle (language servers), Rush (TS extensions) |
| Branching model | GitHub Flow — `main` + one `<major>.<minor>.x` maintenance branch |
| Versioning | SemVer; manual bumps via release pipeline scripts |
| Language server versioning | Bundled with parent extension; no independent release |
| PR triggers | Compile + unit tests + integration tests + quality gates |
| Test levels | Unit, Integration, Tooling/UI E2E |
| Quality gates | SonarQube Cloud (code quality) + Trivy (dependency vulnerabilities) + GitHub Secret Scanning (tokens/secrets) |
| Release pipeline | Nightly (automated on merge to `main`) and Stable/GA (manual `workflow_dispatch` with approval gate) |
| Release cadence | Nightly: continuous; Stable/GA: target every 4–6 weeks, or immediately for critical fixes |
| Release ownership | Designated release owner per stable release; triggers workflow and shepherds the approval gate |
| VS Code extensions | Published to VS Code Marketplace |
| WSO2 Integrator IDE | Published to GitHub Releases |
