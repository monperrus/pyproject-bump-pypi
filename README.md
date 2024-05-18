# pyproject-bump-pypi
A GitHub Action to bump pyproject.toml version number from latest version on Pypi.

Fork of https://github.com/apowis/pyproject-bump-version

## Example Usage:

```yaml
name: "Version bumper"
on:
  pull_request:
    branches:
      - main
    paths:
      - 'python_project/**'
      - './pyproject.toml'

jobs:
  bump-version:
    if: github.event.pull_request.merged == false
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          repository: ${{ github.event.pull_request.head.repo.full_name }}
          ref: ${{ github.event.pull_request.head.ref }}
          fetch-depth: "0"

      - name: Version bumper
        uses: monperrs/pyproject-bump-pypi@v0.0.1
        with:
          file_to_bump: "./pyproject.toml"
          bump_type: "minor"
```

