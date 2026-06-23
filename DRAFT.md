# Awesome Dev Flow :dark_sunglasses:
<!-- markdownlint-disable MD024 -->

## Workflow

### Think 💡

Decide what to build, why it matters, and how delivery will be organized.

#### Discovery

#### Scoping

#### Planning

### Draw 🎨

Define the user experience and the technical blueprint before coding starts.

#### UX/UI Design

#### API Definition

#### Architecture

#### Technical Draft

### Build 🛠️

Implement, connect, and validate the solution until it is release-ready.

#### Development

#### Integration

### Test 🧪

<!-- TODO: Document various kind of tests: regression testing, smoke testing, feature testing, load testing ... -->

### Run 🚀

Launch safely, enable users and teams, and ensure smooth operations.

#### Documentation

#### Deployment

#### Ops & Support

## Tools & Practices

### Manage 🧑‍🍳

#### Ticket Lifecycles

#### Ticket Templates

### Design 🧑‍🎨

#### App Wireframe

Just pen and paper, really.

#### App Prototype

Penpot.

#### API Specs

Insomnia.

### Develop 🧑‍💻

#### IDE

VS Code with some recommended extensions.

#### Environment

ADE -> VS Code devcontainer.

#### Git Branching Strategy

##### Overview

Two branching strategies dominate the industry today. [Trunk-Based Development](https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development) is a modern, lightweight approach where all developers commit frequently to a single long-lived branch (usually `main`), relying on feature flags and a robust automated test suite to keep the codebase always deployable. It excels in SaaS and web application contexts where continuous deployment is the goal. [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow), introduced by Vincent Driessen, takes a more structured approach built around parallel long-lived branches (`main` and `develop`), complemented by short-lived branches for features, releases, and hotfixes. It is designed to support explicit versioning, planned release cycles, and the ability to maintain multiple versions in production simultaneously.

Our product is a multi-application software stack shipped and installed manually on customer hardware. This means we operate with formal release cycles, customers may run different versions in the field for extended periods, and we must be able to deliver patches to older releases without forcing a full upgrade. Trunk-Based Development is optimized for teams deploying to a single, centrally controlled environment many times per day — a model that does not match our delivery constraints. Gitflow, by contrast, was designed precisely for versioned, packaged software: it provides a clear separation between ongoing development work and release-ready code, a dedicated stabilization phase before shipping, and a structured mechanism for hotfixing production versions independently of new feature development.

##### Gitflow Basics

Gitflow revolves around two permanent branches: `main`, which always reflects production-ready code and is tagged with release version numbers, and `develop`, which serves as the integration branch for completed features. Day-to-day development happens on short-lived `feature/*` (e.g. `feature/FLS-2202-add-unknown-status-enum`) branches created from `develop` and merged back into it once complete.

When a release is approaching, a `release/*` branch is cut from `develop` and tagged to allow final QA, bug fixes, and version bumping without blocking ongoing feature work. If there are bugs on the release branch, they get patched in a `bugfix/*` branch. Once validated, the release branch is merged into `main`, tagged (e.g. `v1.2.0`), and merged back into `develop`.

If a critical bug is found in a shipped version, a `hotfix/*` branch is created directly from `main`, fixed, then merged into both `main` and `develop` to keep all branches in sync.

For long-term maintenance of older major versions, `support/*` branches (e.g. `support/1.x`) are created from the relevant version tag, acting as a permanent `main` for that line and serving as the base for any further hotfixes and patch releases (i.e. when you need to release `v1.2.1` but `v2.0.0` is already out).

Make sure to check out [Atlassian's article](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) for a detailed presentation of the Gitflow workflow.

![Gitflow Branch Diagram](assets/git-flow-4.svg)

##### Gitflow for Multi-repos Project

In a multi-repo setup, we have a meta-repo that holds a reference to all the sub-repos in the project (e.g., Git submodules, VCS Tool export). With strict Gitflow, we need to create a new branch in the meta-repo for each new branch in a sub-repo.
Now, features are propagating from the sub-repos, and releases are initiated from the meta-repo, so Gitflow can be simplified a bit. The following sections detail the "simplified" flow for each type of Gitflow action.

###### Feature

Features are used to introduce new changes in the development branch as part of the regular development process.

- Create `feature/<jira-ticket>` branch from sub-repo(s) `develop`.
- _Develop & review the feature._
- Merge `feature/<jira-ticket>` branch into sub-repo(s) `develop`.
- Update sub-repo(s) reference(s) in meta-repo `develop` to target the merge commit on sub-repo(s) `develop`.
- Commit changes to meta-repo `develop` with message `"chore(<jira-ticket>): merge feature"`.

###### Release

In a multi-repo setup, making a release means releasing any changes in the sub-repos with their own individual sub-version numbers, then releasing a new combination of sub-repo versions as a meta-version. Release bumps at least one minor version.

- Create `release/<meta-version>` branch from meta-repo `develop`.
- Generate the changelogs for the meta-repo and each sub-repo with changes.
- Draft the release notes with the version bumps.
- Create `release/<sub-version>` branch from `develop` for each sub-repo with changes.
- Bump the version, if any, in all created `release/*` branches.
- Update sub-repo reference(s) to the last commit, if any, on sub-repo `release/<sub-version>`.
- Commit changes to the meta-repo `release/<meta-version>` with the message `"chore(<jira-ticket>): merge candidate <meta-version>"` and tag `v<meta-version>-rc.1`.
- _Validate the release with the QA full test suite._ If not approved, do bug fixes.
- Edit the release notes.
- Merge sub-repo `release/<sub-version>` into `main` (ff-only) and tag with `v<sub-version>`.
- Merge newly released sub-repos `main` back into `develop`. <!-- No commit required with ff-only: - Update sub-repo reference(s) to the last release tag on sub-repo `main`. - Commit changes to the meta-repo `release/<meta-version>` with the message `"chore: release <meta-version>"` -->
- Merge `release/<meta-version>` into `main` (ff-only) and tag with `v<meta-version>`.
- Merge the newly released meta-repo `main` back into `develop`.

###### Bug Fix

Bug fixes are used to fix bugs against a release branch. The process is the same as for a feature, targeting the `release/*` branch instead of `develop`. A bug fix bumps the `rc` (release candidate) number.

- Create a `bugfix/<jira-ticket>` branch from sub-repo(s) `release/<sub-version>`.
- _Develop & review the bug fix._
- Merge the `bugfix/<jira-ticket>` branch into sub-repo(s) `release/<sub-version>`.
- Merge the `bugfix/<jira-ticket>` branch into sub-repo(s) `develop`.
- Update sub-repo(s) reference(s) to the last commit (if any) on sub-repo(s) `release/<sub-version>`.
- Commit changes to meta-repo `release/<meta-version>` with the message `"chore(<jira-ticket>): merge bugfix"` and tag `v<meta-version>-rc.<rc-number>`.

###### Hot Fix

Hotfixes are used to quickly fix the production branch. From a Gitflow perspective, they combine a feature and a release in one step. A hotfix bumps the patch version.

- Create `hotfix/<meta-version>` branch from meta-repo `main`.
- Create `hotfix/<sub-version>` branch from `main` for each sub-repo with bugs.
- _Develop & review the hotfix._
- Bump the version in all created `hotfix/*` branches.
- Update sub-repo reference(s) in meta-repo `hotfix/<meta-version>` to target the last commit on sub-repo `hotfix/<sub-version>`.
- Commit changes to meta-repo `hotfix/<sub-version>` with message `"chore(<jira-ticket>): merge hotfix <meta-version>"` and tag `v<meta-version>-rc.<rc-number>`.
- _Validate the hotfix with the QA limited test suite._ If not approved, do bug fixes.
- Edit release notes.
- Merge sub-repo `hotfix/<sub-version>` into `main` (ff-only) and tag with `v<sub-version>`.
- Merge newly released sub-repo `main` back into `develop`.<!-- No commit required with ff-only: - Update sub-repo reference(s) to the last release tag on sub-repo `main`. - Commit changes to meta-repo `hotfix/<meta-version>` with message `"chore: patch <meta-version>"` -->
- Merge `release/<meta-version>` into `main` (ff-only) and tag with `v<meta-version>`.
- Merge newly released meta-repo `main` back into `develop`.

###### Support

Support is used to bring fixes to older major releases. It is essentially creating a new base branch for hotfixes. This new base branch will never get merged back into `main`. Support bumps the patch version.

- Create `support/<meta-major-version>.x` branch from the last tag `v<meta-major-version>.*.*` (if it does not exists).
- Create `support/<sub-major-version>.x` branch from the related `v<sub-major-version>.*.*` for each sub-repo that requires changes (if it does not exists).
- Create `hotfix/<meta-version>` branch from meta-repo `support/<meta-major-version>.x`.
- Create `hotfix/<sub-version>` branch from `support/<sub-major-version>.x` for each sub-repo with bugs.
- _Develop & review the hotfix._
- Bump the version in all created `hotfix/*` branches.
- Update sub-repo reference(s) in meta-repo `hotfix/<meta-version>` to target the last commit on sub-repo `hotfix/<sub-version>`.
- Commit changes to meta-repo `hotfix/<sub-version>` with message `"chore(<jira-ticket>): merge hotfix <meta-version>"` and tag `v<meta-version>-rc.<rc-number>`.
- _Validate the hotfix with the QA limited test suite._ If not approved, do bug fixes.
- Edit release notes.
- Merge sub-repo `hotfix/<sub-version>` into `support/<sub-major-version>.x` (ff-only) and tag with `v<sub-version>`.
- Merge meta-repo `hotfix/<meta-version>` into `support/<meta-major-version>.x` (ff-only) and tag with `v<meta-version>`.

##### Gitflow Automation

You've probably noticed that Gitflow, unfortunately, involves some overhead, but that's the cost of having full control and reproducibility over the release process. Instead of losing this very valuable control, we should focus on automating Gitflow.

###### Bitbucket Settings

Bitbucket Server provides several repository settings that help enforce Gitflow discipline across the team. **Branch permissions** (under _Repository Settings → Branch Permissions_) should be configured to protect `main` and `develop`: restrict direct pushes so that all changes must go through a pull request, and require a minimum number of approvals before merging. **Branch name patterns** can be enforced to ensure developers follow the `feature/*`, `release/*`, `bugfix/*`, `hotfix/*`, and `support/*` naming conventions, rejecting branches that do not match. **Merge checks** can be enabled to require successful builds and passing tests (reported via the build status API from your CI tool) before a pull request can be merged. Finally, **default reviewers** can be assigned per branch pattern, automatically adding the relevant team members as reviewers on pull requests targeting `develop` or `main`, ensuring nothing is merged without appropriate oversight.

###### Cocogitto Toolbox

The [Cocogitto](https://docs.cocogitto.io/) Conventional Commits toolbox can be used to create conventional compliant commits with ease, automatically bump versions and generate changelogs with your own custom steps and workflows.

###### Feature Merge Hook

We can set up a webhook in Bitbucket to run a Jenkins job when a sub-repo merges to `develop`:

- clone meta-repo `develop` branch
- update sub-repo ref to latest `develop`
- commit & push changes to meta-repo `develop`

With sub-repos as Git submodules, the job is as simple as:

```bash
#!/bin/bash
# Triggered when a sub-repo merges to develop
git clone url/to/meta-repo
git clone --depth=1 --branch develop --single-branch --recurse-submodule --shallow-submodules
cd path/to/meta-repo
git checkout develop
git submodule update --remote path/to/sub-repo
git add path/to/sub-repo
git commit -m "chore: update sub-repo to latest develop"
git push origin develop
```

We could actually extend this "commit replication logic" for any branches that have the same name in the sub-repo and the meta-repo.

###### Release Creation Script

We can develop a script to help create the release branches on the meta-repo and the sub-repos that have changed.

With sub-repos as Git submodules:

- Creates `release/<meta-version>` from `develop`
- Uses `git diff --submodule` to detect which sub-repos changed
- For each: shows last \<sub-version\>, lists merged PRs, prompts you for the new \<sub-version\>, then creates `release/<sub-version>` from `develop` (if it does not already exist).

#### Git Best Practices

<!-- TODO: List general practice (e.g. branch naming, commit messages, merge strategy, history-rewrite, branch deletion, stale branch policy) -->

##### Branches & Tags

- Branch names are all lowercase and spaces get sanitized as dash (e.g. `FLS-2345 Create my Feature` -> `feature/fls-2345-create-my-feature`).
- `feature/*` and `bugfix/*` branches start with the Jira ticket number (e.g. `feature/fls-2345-create-my-feature`).
- `release/*`, `hotfix/*` and `support/*` branches match the target version number (e.g. `release/2.3.0`, `hotfix/2.3.1`, `support/1.5.12`).
- Version tags always start with `v` (e.g. `v2.3.1`).
- Release candidates (RC) sent for QA validation are tagged with target version number and the RC iteration prefixed with `-rc.` (e.g. `v2.3.1-rc.2` after a first bug fix).

##### Commits

- Commit messages follow the [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification with scope set to the 

##### Merges

- All branche merges must go through a pull request.
- ...

##### Housekeeping

- ...

#### Code Owners

#### Why ?

In large-scale software projects, it is not realistic to assume that every developer on the team knows everything about every piece of code we maintain and evolve. And even if one member of the team actually did, having them be responsible for the entire codebase is not realistic. This leads to slow and/or poor code reviews, frustrated developers, and stressed reviewers.

One solution popular among software teams to ensure fast and high-quality reviews across the entire codebase is to elect code owners. Each code owner is responsible for a given part of the codebase. They do not have to be experts in the assigned code area at the time of election, but they will become so over time.

#### What ?

The code owner responsibilities over the assigned code area include:

- Nurture long-term vision: follow product strategy, anticipate technical challenges, propose evolutions.
- Manage tech debt: monitor code quality reports, organize quality-related work, review PRs.
- Share knowledge: answer designers' and developers' questions, maintain high-level documentation if any.

#### How ?

Bitbucket allows us to define code owners for any path directly in the repository (see [documentation](https://confluence.atlassian.com/bitbucketserver0819/code-owners-1416825984.html)). Code owners will be automatically added to pull requests in the areas of the code base they own.

#### Who ?

> **ToDo**
>
> How we split the code base across the members of the teams still needs to be discussed. But the first step is probably to list all the repositories maintained by the team.
>
> - [ ] List maintained repositories
> - [ ] Make a first ownership split proposal.
> - [ ] Discuss the proposal with the team in a meeting.
>

#### Pull Request

> This only cover pull requests to be submitted for code review. For pull request created for QA testing as part of the release process, see []().

⚠️ All code changes to be merged from a should be covered by a PR.

##### Prerequisites

- Changes satisfy the functional requirements of the related Jira ticket.
- Changes are limited to the related Jira ticket (minor fixes are allowed, but anything that has functional impact must be extracted to a separate ticket).
- Code matches the prescribed style and linting rules.
- Full project build without errors or new warnings.
- Changed packages' unit tests pass successfully.
- No code has any Bug or Code Smell above Major in SonarQube.
- New code has ~80% coverage in SonarQube.

##### Template

```md
## 📖 Context

> Give general context to reviewers and any reader from a distant future who might know nothing about you or what you are doing now. Explain why this PR was necessary in this context. This is basically a summary of the Jira ticket in your own words. You should mention if this PR has merge dependencies with another (e.g., "Do not merge this [this-repo#123]() before [other-repo#34]()").

gRPC-related code is currently split across 3 repos:
- _fls\_grpc\_interface_: proto files & custom compilation instructions for C++ and TypeScript
- _ix\_fls_: ROS2 C++ server implementation, custom compilation instructions for Python, Python scripts for server testing
- _fls-ihm_: TypeScript client implementation, duplicated/branched compilation instructions for TypeScript

Some side effects of this current organization are:
1. Mixing the language-agnostic proto definition of the API with implementation-specific stuff (i.e. CMakeLists.txt, package.json, yarn.lock) in the fls_grpc_interface repo.
2. Having duplicated and stale files in the repos (e.g. compile-proto.sh is in both fls_grpc_interface and fls-ihm).
3. Re-coding existing build logic (e.g. the incremental build in generate_proto.sh already exists in CMake).
4. Re-building the C++ API library on every change to the server implementation (the linking of the C++ generated files is especially slow).
5. Mixing integration test tooling in Python with the server implementation in C++ (implies some kind of dependency where the Python scripts are actually fully independent of the C++ generated code or even protoc built from source for C++).

## 🔨 Changes

> List the changes you made in the code. This can be as simple as the commit list if those were well segmented and described in the first place. Otherwise, this is your chance to make your work path understandable to reviewers without a full Git rebase. If you've made any design choices that were not already specified in the Jira ticket, you should at the very least state them, and ideally justify them. You should also state any limitations if they were not already noted in the Jira ticket.

- Clean up fls_grpc_interface to include only the proto files.
- Build the FLS gRPC C++ API as a separate ROS2 package (much like what we do for ROS messages and services).
- Move the Python test scripts to a separate fls_grpc_tools repo.

## 🏁 Tests

> Explain how to test your changes with steps and expected results (e.g. a screenshot or screen recording). When appropriate, it can be helpful to include a before/after comparison to help the reviewer understand the changes. Since you've already run the SonarQube analysis, it might be a good idea to add a link to the report here.

### Changes in Proto files
Reduced build time by 15% 🙂

#### Before
![Build output on proto file changes before PR](assets/proto_before.png)

#### After
![Build output on proto file changes after PR](assets/proto_after.png)

### Changes in C++ sources
Slashed build time by 50% 🔥

#### Before
![Build output on C++ file changes before PR](assets/cpp_before.png)

#### After
![Build output on C++ file changes after PR](assets/cpp_after.png)
```

##### Review Flow

> **ToDo**
>
> Make a Mermaid chart for the review flow

1. Author submits a PR following the template.
2. Author requests a code review from GitHub Copilot.
3. Author addresses Copilot review comments.
4. Author requests a review from at least one peer (ideally, one who contributed to that area of the code).
5. Peer submits a review within 24 hours (otherwise, croissant for everyone 🥐).
6. Author addresses the peer's review comments (schedule a face-to-face if questions arise).
7. Peer approves the PR (1/2 ✅).
8. Author requests a review from at least one code owner.
9. Code owner submits a review within 24 hours (otherwise, croissant for everyone 🥐).
10. Author addresses the code owner's review comments (schedule a face-to-face if questions arise).
11. Code owner approves the PR (2/2 ✅).
12. Author merges the PR 🚀

#### Releases

<!-- TODO: Checkout how to document release in Jira+Bitbucket. -->

##### Release Schedule

##### Release Flow

##### Release Ticket Template

##### Release Notes Template

##### Release Request Template

#### Coding Style

##### C/C++

Google C++ Style Guide with a few changes as specified in ROS2 Style Guide. Uses CLang Format to apply code style.

Add a note for package.xml and CMakeLists.txt files.

##### Python

Ruff

##### Angular

Prettier (until Biome supports Angular HTML)

##### Protobuf

Buf

#### Linting

##### C/C++

CLang-tidy (to be configured).

##### Python

Ruff & Ty.

##### JavaScript

ESLint (until Biome supports Angular HTML)

##### Protobuf

Buf

#### Unit Testing

##### C/C++

ROS-flavoured GTest and gcovr. Time-based testing is prohibited. Unit tests focus on functional testing. Load/performance testing should be opt-in (i.e. run only on release) to reduce overall unit test run cost.

##### Python

pytest and pytest-cov.

##### JavaScript

Vitest & Playwright.

#### Static Code Analysis

SonarQube with coverage.

### DevOps 👷

#### CI Pipelines

Jenkins.

#### Package Repository

Artifactory.

#### Documentation Page

Cannot use Bitbucket to publish a static website at the moment; need to discuss.

#### Deployment Strategy

Combination of custom bash script, docker-compose and Ansible. Not quite sure what's actually happening here right now.

### Test 🧑‍🔬

Try Zephyr for Jira (paid, but low-cost). Have a look at Qase (full-featured tool for QA).

## People 🧑‍🤝‍🧑

### Roles

> **ToDo**
>
> List the roles involved in software lifecycle along with their responsibilities.

### Team

> **ToDo**
>
> List the persons involved in software lifecycle along with their roles.
