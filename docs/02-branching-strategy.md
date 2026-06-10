# Branching Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/10

This document defines the branching model for all WSO2 Integrator product repos, shared foundation, and product shell.

## Options

| Model | How it works | Pros | Cons |
|---|---|---|---|
| **GitHub Flow with maintenance branch** *(preferred)* | Feature branches off `main`; one `<major>.<minor>.x` patch branch maintained for the current stable release | Clear separation of ongoing work and patch fixes; agent-friendly | Minor additional branch management compared to pure GitHub Flow |
| Trunk-Based Development | Commit directly (or via very short branches) to `main` | Maximum simplicity, fastest integration | Higher risk without a strict feature-flag culture; harder with less experienced contributors |
| GitFlow | Parallel `main` / `develop` + release branches | Well-established for scheduled releases | Heavy branch management across 5 repos; context switching punishes agentic development |

## Recommendation: GitHub Flow with Maintenance Branch

- **`main`** — ongoing feature development targeting the next minor (or major) release. This branch _must_ always be in a releasable state.
- **`<major>.<minor>.x`** (e.g. `1.4.x`) — a single maintenance branch cut from `main` at each GA release. Only patch fixes are backported here; no new features.
- Work happens on short-lived feature branches off `main` (`feat/`, `fix/`, `chore/` prefixes).
- When a new minor GA is released, the previous `<major>.<minor>.x` branch is retired and a new one is cut from the new GA tag.
