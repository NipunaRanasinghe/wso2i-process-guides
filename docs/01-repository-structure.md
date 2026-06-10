# Repository Structure & Build Order

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/10

This document describes the three-layer repository structure of the WSO2 Integrator tooling revamp, the dependency relationships between repos, and the build-order constraints the CI/CD pipelines _must_ respect.

## Repository Layout

The WSO2 Integrator tooling is transitioning from a fragmented multi-repo layout to a clear three-layer structure.

| Layer | Repo(s) | Owns |
|---|---|---|
| **Product repos** | `ballerina-tooling`, `mi-tooling`, `si-tooling` | VS Code extension, welcome page, project creation, product workflows, language server |
| **Shared foundation** | `wso2/vscode-extensions` (transitional name) | Design tokens, icons, fonts, stable helpers/contracts |
| **Product shell** | `product-integrator` | Customised VS Code fork, global config, runtime management, bundling of all extensions |

## Dependency Diagram

```mermaid
graph TD
    SF["<b>shared-foundation</b><br/>(tokens, icons, fonts, helpers)"]

    subgraph ballerina-tooling
        BT_LS["language server (Gradle)"]
        BT_EX["VS Code extension (Rush)"]
        BT_LS --> BT_EX
    end

    subgraph mi-tooling
        MI_LS["language server (Gradle)"]
        MI_EX["VS Code extension (Rush)"]
        MI_LS --> MI_EX
    end

    subgraph si-tooling
        SI_LS["language server (Gradle)"]
        SI_EX["VS Code extension (Rush)"]
        SI_LS --> SI_EX
    end

    PI["<b>product-integrator</b><br/>(WSO2 Integrator IDE shell)"]

    SF --> BT_EX
    SF --> MI_EX
    SF --> SI_EX

    BT_EX --> PI
    MI_EX --> PI
    SI_EX --> PI
```

## Build-Order Constraints

1. **Shared foundation first.** All product-repo extensions consume published packages from the shared foundation. A breaking change in the foundation _must_ be released (or consumed as a pre-release) before dependent extension builds can succeed.

2. **Language server before extension (within a product repo).** Each extension bundles its own language server. The Gradle language-server build _must_ complete and produce an artifact before the Rush extension build packages it.

3. **Extensions before product-integrator.** The `product-integrator` bundling job consumes a specific published version (or local build artifact) of each extension. It does not build extensions from source — it declares them as versioned dependencies. This decouples shell releases from product-repo CI and avoids transitive source coupling.

> **Note:** Each repo's pipeline is self-contained. Cross-repo dependencies are satisfied by versioned artifact references (npm package / VSIX file / Maven artifact), not by triggering upstream pipelines.

## Scope

**Covered by this proposal:** branching model, GitHub Actions pipeline design, testing stages, quality/security gates, SemVer versioning, and the Nightly/Insider vs. Stable/GA release process.

**Not covered:** internal code architecture, runtime behaviour, or the mechanics of the VS Code fork.
