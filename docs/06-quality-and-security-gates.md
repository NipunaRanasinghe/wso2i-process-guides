# Quality & Security Gates

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/10

This document defines the quality and security gates integrated into the PR pipeline and the rationale behind each.

| Requirement | Tool | Notes |
|---|---|---|
| Code quality (coverage, duplication, complexity) | SonarQube Cloud | GitHub Actions step; posts results to the PR; free tier to start |
| Dependency vulnerability scanning | Trivy | GitHub Actions step; already configured in all repos |
| Secret / token detection | GitHub Secret Scanning | Enabled at repo level (Settings → Security); no pipeline step needed; free for public repos |
