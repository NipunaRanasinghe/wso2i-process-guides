# WSO2 Integrator - Architecture & Process Guidelines
 
The WSO2 Integrator tooling spans five GitHub repositories, each containing multiple components that together form the integrated development experience. This document defines the core aspects of the architecture and development process that govern the development and maintenance of the product tooling. 

This document serves as a single source of truth for the processes and practices that all contributors to the WSO2 Integrator tooling are expected to follow. Each topic is covered in its own dedicated document linked below.


| # | Document | Summary |
|---|---|---|
| 1 | [Component Architecture](01-component-architecture.md) | Repo layer layout, dependency diagram, and build-order constraints |
| 2 | [Branching Strategy](02-branching-strategy.md) | `main` + `<major>.<minor>.x` maintenance branch model |
| 3 | [Versioning Strategy](03-versioning-strategy.md) | SemVer and language server versioning |
| 4 | [CI/CD Pipelines](04-cicd-pipelines.md) | PR pipeline structure, release tracks, and quality gates |
| 5 | [Testing Strategy](05-testing-strategy.md) | Test types, pipeline stages, and blocking gates |
| 6 | [Quality & Security Gates](06-quality-and-security-gates.md) | GHAS + SonarQube Cloud recommendation |
| 7 | [Release Process](07-release-process.md) | Release types, cadence, ownership, and step-by-step release processes |
