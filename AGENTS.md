# `problem4j-workflows` - Agent Instructions

Reusable GitHub Actions workflows shared across all `problem4j` repositories.

## Project Layout

| Path                 | Contents                                                            |
|----------------------|---------------------------------------------------------------------|
| `.github/workflows/` | Reusable workflow definitions and `release-workflows.yml` (release) |
| `README.md`          | Usage and release documentation                                     |

## Agent Rules

- This repository contains only YAML workflow files and documentation - there is no build system.
- When editing workflows, keep caller workflows thin: triggers, matrix, and `uses:` delegation only.
- All build logic lives in `.github/workflows/` - callers in other repositories must not duplicate steps.
- Workflows are versioned with `vX.Y.Z` tags and a major version tag `vX`, moved to the highest release of that major
  version - both created by `release-workflows.yml`. Callers reference `vX` tags. Breaking changes for callers (renamed
  workflows, inputs or secrets) require a new major version.
