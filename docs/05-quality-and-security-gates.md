# Quality & Security Gates

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/10

This document defines the code quality and security scanning tools integrated into the PR pipeline and the rationale for the recommended stack.

## Options

| Tool | Category | Integration Effort | Cost Model |
|---|---|---|---|
| **GitHub Advanced Security (GHAS)** *(preferred)* | Code scanning (CodeQL) + secret scanning + dependency review | Native to GitHub, zero config for public repos | Included with GitHub Enterprise / billed per committer |
| SonarQube Cloud | Code quality + coverage gate | Requires SonarQube project setup per repo | Free tier available; paid for private repos at scale |
| Mend (WhiteSource) | SCA / dependency vulnerability scanning | GitHub Action + Mend account | Commercial |
| Veracode | SAST + SCA | Separate scan submission, longer feedback loop | Commercial |

## Recommendation: GHAS + SonarQube Cloud

- **GHAS** for secret scanning, dependency review (in PR), and CodeQL static analysis — covers security posture with near-zero pipeline overhead.
- **SonarQube Cloud** (free tier to start) for code quality gates (coverage thresholds, duplication, complexity) — gives actionable PR feedback on quality regressions.

Both tools integrate as GitHub Actions steps and post results directly to the PR, fitting the PR pipeline described in [CI/CD Pipeline Design](03-cicd-pipeline-design.md).

> **Note:** Mend or Veracode _should_ be evaluated if compliance requirements surface in the future.
