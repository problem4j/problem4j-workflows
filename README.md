# Problem4J Workflows

Reusable GitHub Actions workflows shared across all [`problem4j`](https://github.com/problem4j)
repositories.

## Table of Contents

- [Versioning](#versioning)
- [Workflows](#workflows)
  - [ci.yml](#ciyml)
  - [gradle-dependency-submission.yml](#gradle-dependency-submissionyml)
  - [release.yml](#releaseyml)

## Versioning

Workflows are versioned with immutable `vX.Y.Z` Git tags. Tags are never moved, so callers must reference an exact
release tag (e.g. `@v1.0.0`). A new major version means breaking changes for callers, such as renamed or removed
workflows, inputs or secrets.

## Workflows

### `ci.yml`

See workflow - [link](.github/workflows/ci.yml).

Builds the project with Gradle. Code style check runs as a separate, conditional step before the build, which allows
enabling it only on selected Java versions (formatters such as `google-java-format` resolve to different versions
depending on Java version, which may result in different formatting). The code style task is excluded from the build
step.

| Input                | Type      | Required | Default         | Description                                         |
|----------------------|-----------|----------|-----------------|-----------------------------------------------------|
| `java-version`       | `number`  | no       | `25`            | Java version to build with (e.g. `17`, `21`, `25`). |
| `code-style-enabled` | `boolean` | no       | `true`          | Whether to run code style check step.               |
| `code-style-task`    | `string`  | no       | `spotlessCheck` | Gradle task verifying code style.                   |
| `codecov-enabled`    | `boolean` | no       | `false`         | Whether to upload JaCoCo report to Codecov.         |
| `codecov-files`      | `string`  | no       | `""`            | JaCoCo files to upload to Codecov.                  |

**Usage:**

```yaml
jobs:
  build:
    name: Build (Java ${{ matrix.java }})
    strategy:
      fail-fast: false
      matrix:
        java: [17, 21, 25]
    uses: problem4j/problem4j-workflows/.github/workflows/ci.yml@v1.0.0
    with:
      java-version: ${{ matrix.java }}
      code-style-enabled: ${{ matrix.java >= 21 }}
      codecov-enabled: ${{ matrix.java == 25 }}
      codecov-files: "build/reports/jacoco/test/jacocoTestReport.xml"
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

---

### `gradle-dependency-submission.yml`

See workflow - [link](.github/workflows/gradle-dependency-submission.yml).

Generates and submits the Gradle dependency graph to GitHub for dependency review and Dependabot alerts.

| Input          | Type     | Required | Default | Description          |
|----------------|----------|----------|---------|----------------------|
| `java-version` | `number` | no       | `25`    | Java version to use. |

**Usage:**

```yaml
jobs:
  dependency-submission:
    name: Dependency Submission
    permissions:
      contents: write
    uses: problem4j/problem4j-workflows/.github/workflows/gradle-dependency-submission.yml@v1.0.0
```

---

### `release.yml`

See workflow - [link](.github/workflows/release.yml).

Publishes a release to Maven Central using the [nmcp](https://github.com/GradleUp/nmcp) Gradle 
plugin. Triggered by a version tag push; extracts the version from the tag name (strips the leading `v`).

| Input          | Type     | Required | Default | Description                 |
|----------------|----------|----------|---------|-----------------------------|
| `java-version` | `number` | no       | `25`    | Java version to use.        |
| `publish-task` | `string` | yes      | —       | Gradle publish task to run. |

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
    uses: problem4j/problem4j-workflows/.github/workflows/release.yml@v1.0.0
    with:
      publish-task: publishAllPublicationsToCentralPortal
    secrets: inherit
```

Setting `secrets: inherit` forwards all secrets from the calling workflow to the reusable workflow
automatically, without mapping each one individually.
