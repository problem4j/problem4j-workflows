# Problem4J Workflows

Reusable GitHub Actions workflows shared across all [`problem4j`](https://github.com/problem4j)
repositories.

## Table of Contents

- [Workflows](#workflows)
  - [gradle-build.yml](#gradle-buildyml)
  - [gradle-dependency-submission.yml](#gradle-dependency-submissionyml)
  - [gradle-publish-release.yml](#gradle-publish-releaseyml)

## Workflows

### `gradle-build.yml`

See workflow - [link](.github/workflows/gradle-build.yml).

Builds the project with Gradle across multiple Java versions.

| Input          | Type     | Required | Description                                         |
|----------------|----------|----------|-----------------------------------------------------|
| `java-version` | `number` | yes      | Java version to build with (e.g. `17`, `21`, `25`). |

**Usage:**

```yaml
jobs:
  build:
    name: Build (Java ${{ inputs.java-version }})
    strategy:
      fail-fast: false
      matrix:
        java: [17, 21, 25]
    uses: problem4j/problem4j-workflows/.github/workflows/gradle-build.yml@main
    with:
      java-version: ${{ matrix.java }}
```

---

### `gradle-dependency-submission.yml`

See workflow - [link](.github/workflows/gradle-dependency-submission.yml).

Generates and submits the Gradle dependency graph to GitHub for dependency review and Dependabot alerts.

| Input          | Type     | Required | Description               |
|----------------|----------|----------|---------------------------|
| `java-version` | `number` | yes      | Java version to use.      |

**Usage:**

```yaml
jobs:
  dependency-submission:
    name: Dependency Submission
    permissions:
      contents: write
    uses: problem4j/problem4j-workflows/.github/workflows/gradle-dependency-submission.yml@main
    with:
      java-version: 17
```

---

### `gradle-publish-release.yml`

See workflow - [link](.github/workflows/gradle-publish-release.yml).

Publishes a release to Maven Central using the [nmcp](https://github.com/GradleUp/nmcp) Gradle 
plugin. Triggered by a version tag push; extracts the version from the tag name (strips the leading `v`).

| Input          | Type     | Required | Description                 |
|----------------|----------|----------|-----------------------------|
| `java-version` | `number` | yes      | Java version to use.        |
| `publish-task` | `string` | yes      | Gradle publish task to run. |

| Secret                | Required | Description                                 |
|-----------------------|----------|---------------------------------------------|
| `SIGNING_KEY`         | yes      | GPG signing key                             |
| `SIGNING_PASSWORD`    | yes      | GPG signing password                        |
| `PUBLISHING_USERNAME` | yes      | Maven Central (Sonatype) tokenized username |
| `PUBLISHING_PASSWORD` | yes      | Maven Central (Sonatype) tokenized password |

The `publish-task` depends on the repository structure:
- Single-module repositories use `publishAllPublicationsToCentralPortal` (e.g. `problem4j-core`).
- Multi-module repositories with an aggregation publication use `publishAggregationToCentralPortal`
  (e.g. `problem4j-spring`).

**Usage:**

```yaml
jobs:
  publish-release:
    name: Publish Release
    uses: problem4j/problem4j-workflows/.github/workflows/gradle-publish-release.yml@main
    with:
      publish-task: publishAllPublicationsToCentralPortal
    secrets: inherit
```

Setting `secrets: inherit` forwards all secrets from the calling workflow to the reusable workflow
automatically, without mapping each one individually.
