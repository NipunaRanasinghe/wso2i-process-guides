I need to create a document (proposal) to define and cover all CI/CD, release, versioning and other related pipelines workflows and processes related aspects along with the WSO2 Integrator new structure. Proposed repo (component) structure for the WSO2 Integrator is in the proposed-repo-structure.png file.

Some additional context are attached below in a question-answer format. you can use that to fill the gaps in the proposal.

# Questions

### 1. Repository & Architecture Structure

* **Repo Type:** Is the new structure a monorepo (e.g., managed via Turbo, Nx, Maven multi-module) or a multi-repo setup?
* **Components:** What specific artifacts are built from this repo? (e.g., VS Code extensions, IntelliJ plugins, CLI tools, shared library modules, Docker images, or runtime components?)
* **Dependencies:** Are there strict build-order dependencies between these components that the CI/CD pipeline needs to respect?

### 2. CI/CD Platform & Tools

* **Platform:** What CI/CD platform are you planning to use? (e.g., GitHub Actions, Jenkins, GitLab CI)
* **Build Tools:** What are the primary build systems used across the components? (e.g., Maven, Gradle, npm/pnpm/yarn, Go)

### 3. Branching Strategy & Pipeline Triggers

* **Branching Model:** What Git branching strategy do you follow? (e.g., Trunk-Based Development, GitHub Flow with feature branches, or GitFlow with `main`/`develop` branches?)
* **Pipeline Triggers:** * What should happen on a Pull Request (PR)? (e.g., compile, unit tests, linters, SAST scanning)
* What should happen when code is merged into the main branch? (e.g., snapshot/nightly build, automatic deployment to a staging registry)



### 4. Testing & Quality Gates

* **Test Suites:** What levels of testing need to be integrated into the automated pipeline? (e.g., Unit tests, Integration tests, Tooling/UI E2E tests, backward compatibility tests)
* **Quality/Security Gates:** Do you require integration with tools like SonarQube, Veracode, Mend (WhiteSource), or GitHub Advanced Security?

### 5. Release & Versioning Strategy

* **Versioning:** Do you follow Semantic Versioning (SemVer)? How are versions managed and bumped? (e.g., manual updates, or automated using tools like *Release Please* or *Semantic Release* based on Conventional Commits?)
* **Release Artifact Registries:** Where do the final release artifacts need to be published?
* *VS Code Marketplace / Open VSX?*
* *JetBrains Marketplace?*
* *Maven Central / Nexus?*
* *npm registry / GitHub Packages?*
* *Docker Hub / Quay.io?*


* **Changelogs:** Should the release pipeline automatically generate changelogs/release notes?

### 6. Environment & Promotion Path

* **Environments:** Do you maintain distinct environments for tooling validation (e.g., Nightly/Insider preview channels vs. Stable/GA channels)?
* **Approval Approvals:** Do production releases require manual approval gates within the CI/CD pipeline, or are they fully automated upon tagging a release?


# ANSWERS


1. 
repo type:  you can see that from the proposed structure image I attached
Components: 4 vscode extensions + the WSO2 integrator IDE (a vscode fork). apart from that we need to decide if we need independent releases and versioning for 3 language servers inside the repos 
Dependencies: yes. derive that from the attached image. 

2. 
Platform: GH actions
Build Tools: gradle and rush

3. 
Branching Model: <you can suggest based on the proposed system>
Pipeline Triggers: compile, tests

4. 
Test Suites: Unit tests, Integration tests, Tooling/UI E2E tests
Quality/Security Gates: <you can suggest based on the proposed system>

5. 
Versioning: SemVer. version bumps can be planned in an automated manner
Release Artifact Registries: all vscode plugins should go to VS Code Marketplace, and should be packed within our WSO2 integrator IDE. wso2 integrator IDE will be published to GH release artifacts https://github.com/wso2/product-integrator/releases
Changelogs: better if we can do that 

6. 
Environments: yes, better to have distinct workflows Nightly/Insider preview channels vs. Stable/GA channels. 
Approvals: do you think its better to have manual approval gates? do with your suggestion


6. 