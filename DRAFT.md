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

#### QA Testing

### Run 🚀

Launch safely, enable users and teams, and ensure smooth operations.

#### Documentation

#### Release

#### Operate & Support

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

#### Git Flow

#### Code Ownership

#### Pull Request

All code changes should be covered by a PR.

##### Prerequisites

- Changes satisfy the functional requirements of the related Jira ticket.
- Changes are limited to the related Jira ticket (minor fixes are allowed, but anything that has functional impact must be extracted to a separate ticket).
- Code matches the prescribed style and linting rules.
- Full project build without errors or new warnings.
- Changed packages' unit tests pass successfully.
- No code has any Bug or Code Smell above Major in SonarQube.
- New code has ~80% coverage in SonarQube.

##### Template

<!-- TODO: Add examples to PR Template-->

```md
## Context

> Give general context to reviewers and any reader from a distant future who might know nothing about you or what you are doing now. Explain why this PR was necessary in this context. This is basically a summary of the Jira ticket in your own words. You should mention if this PR has merge dependencies with another (e.g., "Do not merge this [this-repo#123]() before [other-repo#34]()").

## Changes

> List the changes you made in the code. This can be as simple as the commit list if those were well segmented and described in the first place. Otherwise, this is your chance to make your work path understandable to reviewers without a full Git rebase. If you've made any design choices that were not already specified in the Jira ticket, you should at the very least state them, and ideally justify them. You should also state any limitations if they were not already noted in the Jira ticket.

## Tests

> Explain how to test your changes with steps and expected results (e.g., a screenshot or screen recording). When appropriate, it can be helpful to include a before/after comparison to help the reviewer understand the changes. Since you've already run the SonarQube analysis, it might be a good idea to add a link to the report here.
```

##### Flow

1. Author submits a PR following the template.
2. Author requests a code review from GitHub Copilot.
3. Author addresses Copilot review comments.
4. Author requests a review from at least one peer (ideally, one who contributed to that area of the code).
5. Peer submits a review within 24 hours (otherwise, croissant for everyone 🥐).
6. Author addresses the peer's review comments (schedule a face-to-face if questions arise).
7. Peer approves the PR.
8. Author requests a review from at least one code owner.
9. Code owner submits a review within 24 hours (otherwise, croissant for everyone 🥐).
10. Author addresses the code owner's review comments (schedule a face-to-face if questions arise).
11. Code owner approves the PR.
12. Author merges the PR.

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

#### Code Quality Analysis

SonarQube with coverage.

#### CI/CD Pipelines

Jenkins.

#### Package Repository

Artifactory.

#### Documentation Repository

Cannot use Bitbucket to publish a static website at the moment; need to discuss.

##### Test 🧑‍🔬

Try Zephyr for Jira (paid, but low-cost). Have a look at Qase (full-featured tool for QA).
