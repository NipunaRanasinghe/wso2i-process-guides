# Quality & Security Gates

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/10

This document defines the code quality gates integrated into the PR pipeline and the rationale for the recommended stack.

## Options

| Tool | Category | Integration Effort | Cost Model |
|---|---|---|---|
| **SonarQube Cloud** *(preferred)* | Code quality + coverage gate | Requires SonarQube project setup per repo | Free tier available; paid for private repos at scale |
| Veracode | SAST + SCA | Separate scan submission, longer feedback loop | Commercial |

## Recommendation: SonarQube Cloud

- **SonarQube Cloud** (free tier to start) for code quality gates (coverage thresholds, duplication, complexity) — gives actionable PR feedback on quality regressions.

SonarQube Cloud integrates as a GitHub Actions step and posts results directly to the PR, fitting the PR pipeline described in [CI/CD Pipeline Design](04-cicd-pipeline-design.md).
