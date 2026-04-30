# `problem4j-workflows` - Agent Instructions

Reusable GitHub Actions workflows shared across all `problem4j` repositories.

## Project Layout

| Path                       | Contents                        |
|----------------------------|---------------------------------|
| `.github/workflows/`       | Reusable workflow definitions   |
| `README.md`                | Usage documentation             |

## Agent Rules

- This repository contains only YAML workflow files and documentation - there is no build system.
- When editing workflows, keep caller workflows thin: triggers, matrix, and `uses:` delegation only.
- All build logic lives in `.github/workflows/` - callers in other repositories must not duplicate steps.
