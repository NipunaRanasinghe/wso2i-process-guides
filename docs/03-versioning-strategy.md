# Versioning Strategy

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/10

This document defines the versioning scheme applied across all WSO2 Integrator repos and the decision on language server versioning.

## SemVer

All repos _must_ follow [Semantic Versioning (SemVer)](https://semver.org/). Version bumps in manifests (`package.json`, `pom.xml`) are updated as part of the release workflow, with automated version increment scripts invoked by the release pipeline to reduce human error.

## Language Server Versioning

Language servers are versioned and released together with their parent extension — there is no independent language server release. Each product version (e.g. `ballerina-tooling 1.4.0`) includes a specific language server build; consumers always receive a tested, matched pair.

> **Note:** If an external consumer of a language server emerges in the future, the repo structure supports promoting it to an independent release without structural change.
