# Component Architecture

_Authors_: @NipunaRanasinghe \
_Reviewers_: \
_Created_: 2026/06/09 \
_Updated_: 2026/06/11

This document defines the component layout, dependency relationships, and build order across the WSO2 Integrator tooling repos.

## Repository Layout

| Layer | Repo(s) | Owns |
|---|---|---|
| **Shared UI toolkit** | [`vscode-extensions`](https://github.com/wso2/vscode-extensions) | icons, fonts, helpers, contracts |
| **Product tooling** | [`ballerina-tooling`](https://github.com/wso2/ballerina-vscode/), [`mi-tooling`](https://github.com/wso2/mi-vscode), [`si-tooling`](https://github.com/siddhi-io/siddhi-plugin-vscode/) | VS Code extensions, language servers, and product-specific components |
| **Product distribution** | [`product-integrator`](https://github.com/wso2/product-integrator/) | WSO2 Integrator VS Code Extension (orchestrating VS Code plugin) + WSO2 Integrator IDE (customised VS Code fork, global config, runtime management) |

> **Note:** This document describes the _target_ architecture. Repositories are currently in various states of migration toward this layout, and some cross-repo dependencies are not yet aligned with it. The goal is to converge on this architecture across all repos.

## Dependency Diagram

The diagram below shows the build-time dependencies between components across the repos.

An arrow from A to B means B consumes A, either by:
- bundling it into its artifact (e.g. a language server into its parent extension) or
- declaring it as a versioned dependency (e.g. toolkit packages consumed by extensions).

```mermaid
graph BT
    subgraph VSC["vscode-extensions"]
        UIT["Common UI Toolkit"]
    end

    subgraph BALL["ballerina-tooling"]
        BT_LS["Ballerina Language Server"]
        BT_GR["Grammar"]
        BT_EX["Ballerina VS Code Extension"]
        BT_LS --> BT_EX
        BT_GR --> BT_EX
    end

    subgraph MI["mi-tooling"]
        MI_LS["MI Language Server"]
        MI_EX["MI VS Code Extension"]
        MI_LS --> MI_EX
    end

    subgraph SI["si-tooling"]
        SI_LS["SI Language Server"]
        SI_EX["SI VS Code Extension"]
        SI_LS --> SI_EX
    end

    subgraph PROD["product-integrator"]
        PI_EX["WSO2 Integrator VS Code Extension"]
        PI_IDE["WSO2 Integrator IDE"]
        PI_EX --> PI_IDE
    end

    UIT --> BT_EX
    UIT --> MI_EX
    UIT --> SI_EX
    UIT --> PI_EX

    BT_EX --> PI_EX
    MI_EX --> PI_EX
    SI_EX --> PI_EX
```

## Build Order

Builds _must_ respect the following dependency order.

1. **Shared UI toolkit.** Product-repo extensions consume published packages from `vscode-extensions`. A breaking change in the shared UI toolkit _must_ be released before any dependent extension build can proceed (see [Cross-Repo Version Bumps](03-versioning-strategy.md#cross-repo-version-bumps)).

2. **Language server before extension.** Each VS Code extension bundles its own language server. The Gradle build _must_ produce an artifact before the Rush build can package it.

3. **Product extensions before `product-integrator`.** The WSO2 Integrator VS Code Extension declares each product extension as a versioned dependency — it does not build them from source. The WSO2 Integrator IDE in turn bundles the WSO2 Integrator VS Code Extension. This keeps the product distribution decoupled from product-repo CI.

> **Note:** Each repo's pipeline is self-contained. Cross-repo dependencies are resolved through versioned artifact references (`npm` package, VSIX, or Maven artifact) — not by triggering upstream pipelines.
