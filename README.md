# pyproject-bump-pypi

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A GitHub Action that automatically bumps the version in your `pyproject.toml` by comparing the **latest published version on PyPI** with your local file — so your next release is always one step ahead of what is already published.

## Description

`pyproject-bump-pypi` reads the package name from your `pyproject.toml`, fetches the latest published version from PyPI, computes the next version according to the requested bump type (`major`, `minor`, or `micro`), and writes it back into `pyproject.toml`. Because the new version is derived from the PyPI version rather than the local one, the action stays in sync with reality even if the file was edited manually between releases.

## How It Works

1. The package name is read from `[project].name` in `pyproject.toml`.
2. The latest published version is fetched from `https://pypi.org/pypi/<name>/json`.
3. The next version is computed by incrementing the PyPI version according to `bump_type`.
4. The result is written back into `[project].version` in `pyproject.toml`.

**First-publish edge case:** if the package has never been published to PyPI (HTTP 404), the action uses the local version as the baseline and skips bumping, so a brand-new project is not accidentally incremented before its first release.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `file_to_bump` | ✅ | Path to the `pyproject.toml` file to update (e.g. `"./pyproject.toml"`). |
| `bump_type` | ✅ | Version component to increment. Allowed values: `major`, `minor`, `micro`. |

## Outputs

| Output | Description |
|--------|-------------|
| `bumped` | `false` if no bump was performed (e.g. first publish). Otherwise the commit message string, e.g. `"Bumped version from 1.2.0 to 1.3.0"`. |

## Usage

```yaml
name: Bump version on push to main

on:
  push:
    branches:
      - main

jobs:
  bump-version:
    runs-on: ubuntu-latest
    # Allow the job to push back the updated pyproject.toml
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          # Fetch the full history so the commit can be pushed
          fetch-depth: 0

      - name: Bump pyproject.toml version
        id: bump
        uses: monperrus/pyproject-bump-pypi@last
        with:
          file_to_bump: "./pyproject.toml"
          bump_type: "minor"   # or "major" / "micro"

      # Commit the updated pyproject.toml back to the branch.
      # Remove this step if you handle the commit elsewhere.
      - name: Commit version bump
        if: steps.bump.outputs.bumped != 'false'
        uses: EndBug/add-and-commit@v9
        with:
          message: ${{ steps.bump.outputs.bumped }}
          add: "./pyproject.toml"
```

> **Permissions note:** The workflow needs `contents: write` (or an equivalent token) to push the commit. When using the default `GITHUB_TOKEN` this is sufficient for most repositories; for forks you may need to supply a personal access token via `token:` in the checkout step.

## Requirements

Your `pyproject.toml` must declare both fields in the `[project]` table (standard [PEP 621](https://peps.python.org/pep-0621/) format):

```toml
[project]
name = "your-package-name"
version = "1.2.3"
```

## License

MIT. Forked from [apowis/pyproject-bump-version](https://github.com/apowis/pyproject-bump-version).

