# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This workspace contains the **WSO2 Integrator Architecture & Process Guidelines** proposal for the tooling architecture revamp. There is no runnable code — the deliverable is a set of Markdown documents.

## Files

- `WSO2 Integrator Architecture Revamp proposal.md` — Architecture context document. Describes existing issues, objectives, ownership split, and outstanding steps.

## Proposal Documents

The proposal is split into `docs/`, one document per topic. See `docs/README.md` for the full index.

| File | Topic |
|---|---|
| `docs/component-architecture.md` | Component ownership, dependency diagram, and build-order constraints |
| `docs/branching-strategy.md` | `main` + `<major>.<minor>.x` maintenance branch model |
| `docs/versioning-strategy.md` | SemVer, language server versioning |
| `docs/cicd-pipelines.md` | PR pipeline structure and release tracks |
| `docs/testing-strategy.md` | Test types, pipeline stages, blocking gates |
| `docs/quality-and-security-gates.md` | GHAS + SonarQube Cloud |
| `docs/release-process/` | Release cadence, ownership, checklist, post-release steps |

`.claude/writing-style.md` — Style guide for all documents in this workspace, derived from the ballerina-platform/ballerina-library docs corpus. Apply it when creating or editing any doc.

`.claude/repo-state-snapshot.md` — Observed state of all five repos as of 2026/06/12. Use this as the baseline when checking for drift between the proposal docs and reality.

## Repos

Use these to validate and verify any existing or future claims in the proposal docs.

| Repo (proposal name) | GitHub URL | Notes |
|---|---|---|
| `product-integrator` | https://github.com/wso2/product-integrator/ | WSO2 Integrator VS Code Extension + WSO2 Integrator IDE |
| `ballerina-tooling` | https://github.com/wso2/ballerina-vscode/ | Pending rename to `ballerina-tooling` |
| `mi-tooling` | https://github.com/wso2/mi-vscode | Pending rename to `mi-tooling` |
| `vscode-extensions` | https://github.com/wso2/vscode-extensions | Shared UI libraries |
| `si-tooling` | https://github.com/siddhi-io/siddhi-plugin-vscode/ | Pending rename to `si-tooling` |

## Key Decisions (resolved)

| Topic | Decision |
|---|---|
| CI/CD platform | GitHub Actions |
| Build tools | Gradle (language servers), Rush (TS extensions) |
| Branching model | GitHub Flow — `main` + one `<major>.<minor>.x` maintenance branch |
| Versioning | SemVer; manual bumps via release pipeline scripts |
| Language server versioning | Bundled with parent extension; no independent release |
| Shared UI libraries consumption | Git submodule of `vscode-extensions`; built from source; no independent libraries release; upstream-first (changes land on `main`; consumers move the submodule pointer forward) |
| PR triggers | Compile + unit tests + integration tests + quality gates |
| Test levels | Unit, Integration, Tooling/UI E2E |
| Quality gates | SonarQube Cloud (code quality) + Trivy (dependency vulnerabilities) + GitHub Secret Scanning (tokens/secrets) |
| Release pipeline | Nightly (automated on a daily schedule) and Stable/GA (manual `workflow_dispatch` with approval gate) |
| Release cadence | Nightly: continuous; Stable/GA: target every 4–6 weeks, or immediately for critical fixes |
| Release ownership | Designated release owner per stable release; triggers workflow and obtains the approval-gate sign-off |
| VS Code extensions | Published to VS Code Marketplace |
| WSO2 Integrator IDE | Published to GitHub Releases |
