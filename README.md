# Semantic Versioning Action

[![Test and Release](https://github.com/TheKathan/semantic-versioning/actions/workflows/test.yml/badge.svg)](https://github.com/TheKathan/semantic-versioning/actions/workflows/test.yml)

Automatic semantic versioning for GitHub repositories based on branch naming conventions.

## Features

- 🎯 **Main Branch Versioning**: Determines version bump from merged PR source branch
- 🚀 **Pre-release Support**: Optional pre-release versions on feature/fix branches
- 🔄 **Smart Detection**: Prevents duplicate tags on same commit
- 📦 **Reusable**: Works across all your repositories

## Usage

### Basic (Main Branch Only)

```yaml
name: Release
on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create Version
        uses: TheKathan/semantic-versioning@v1
        with:
          major-version: "1"
```

### With Pre-release Support

```yaml
name: Version
on:
  push:
    branches: [main, 'feature/**', 'fix/**']

jobs:
  version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Create Version
        id: version
        uses: TheKathan/semantic-versioning@v1
        with:
          major-version: "1"
          enable-prerelease: "true"

      - name: Create Release
        if: steps.version.outputs.is-prerelease == 'false'
        run: |
          gh release create ${{ steps.version.outputs.version }} --generate-notes
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `major-version` | Major version number | No | `1` |
| `enable-prerelease` | Enable pre-release versioning | No | `false` |
| `github-token` | GitHub token for API access | No | `${{ github.token }}` |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `version` | Calculated semantic version | `v1.2.3` |
| `is-prerelease` | Whether this is a pre-release | `true` / `false` |

## Branch Patterns

### Main Branch (default mode)

When a PR is merged to `main`:

| Source Branch | Version Bump | Example |
|---------------|--------------|---------|
| `feature/*` or `feat/*` | Minor (1.1.0 → 1.2.0) | `v1.2.0` |
| `fix/*`, `hotfix/*`, `chore/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |

### Pre-release Mode (`enable-prerelease: true`)

On feature/fix branches:

| Branch Pattern | Version Format | Example |
|----------------|----------------|---------|
| `feature/*` or `feat/*` | `v{major}.{minor}.{patch}-alpha.{n}` | `v1.2.0-alpha.1` |
| `fix/*` or `hotfix/*` | `v{major}.{minor}.{patch}-fix.{n}` | `v1.1.1-fix.1` |
| `chore/*` | `v{major}.{minor}.{patch}-chore.{n}` | `v1.1.1-chore.1` |

## Examples

### Example 1: Feature Development

```yaml
# .github/workflows/version.yml
name: Version
on:
  push:
    branches: [main, 'feature/**']

jobs:
  version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: TheKathan/semantic-versioning@v1
        with:
          enable-prerelease: "true"
```

**Result:**
- Push to `feature/new-login` → Creates `v1.2.0-alpha.1`
- Push again → Creates `v1.2.0-alpha.2`
- Merge to `main` → Creates `v1.2.0`

### Example 2: Release Only

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Version
        id: version
        uses: TheKathan/semantic-versioning@v1

      - name: Release
        run: |
          gh release create ${{ steps.version.outputs.version }} \
            --title "Release ${{ steps.version.outputs.version }}" \
            --generate-notes
```

## How It Works

1. **Tag Detection**: Checks if current commit already has a version tag
2. **Base Version**: Finds latest version tag matching major version
3. **Branch Analysis**:
   - Main: Detects merged PR source branch
   - Pre-release: Uses current branch pattern
4. **Version Calculation**: Applies semantic versioning rules
5. **Tag Creation**: Creates and pushes new version tag

## License

MIT

## Contributing

Issues and PRs welcome at [TheKathan/semantic-versioning](https://github.com/TheKathan/semantic-versioning)
