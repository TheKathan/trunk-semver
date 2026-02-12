# Semantic Versioning Action

[![Test and Release](https://github.com/TheKathan/semantic-versioning/actions/workflows/test.yml/badge.svg)](https://github.com/TheKathan/semantic-versioning/actions/workflows/test.yml)
[![version](https://img.shields.io/github/v/release/TheKathan/semantic-versioning?label=version)](https://github.com/TheKathan/semantic-versioning/releases/latest)
[![License](https://img.shields.io/github/license/TheKathan/semantic-versioning)](LICENSE)
[![Issues](https://img.shields.io/github/issues/TheKathan/semantic-versioning)](https://github.com/TheKathan/semantic-versioning/issues)
[![PRs](https://img.shields.io/github/issues-pr/TheKathan/semantic-versioning)](https://github.com/TheKathan/semantic-versioning/pulls)

Automatically version your releases based on branch naming conventions. Works on main branch merges with optional pre-release support.

> Testing PR comment and job summary features.

## Quick Start

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

      - uses: TheKathan/semantic-versioning@v1
        with:
          major-version: "1"
```

That's it. When you merge a PR to main, it figures out the version bump from your branch name and tags it automatically.

## How It Works

The action looks at your merged PR's source branch and bumps the version accordingly:
- `feature/*` or `feat/*` → bumps minor version (1.0.0 → 1.1.0)
- Everything else → bumps patch version (1.0.0 → 1.0.1)

It won't create duplicate tags if the commit already has one, and you can set your own major version.

## Branch Patterns

### Main Branch (default mode)

When a PR is merged to `main`:

| Source Branch | Version Bump | Example |
|---------------|--------------|---------|
| `feature/*`, `feat/*` | Minor (1.1.0 → 1.2.0) | `v1.2.0` |
| `fix/*`, `hotfix/*`, `bugfix/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |
| `chore/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |
| `docs/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |
| `refactor/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |
| `perf/*`, `performance/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |
| `test/*`, `tests/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |
| `build/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |
| `ci/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |
| `style/*` | Patch (1.1.0 → 1.1.1) | `v1.1.1` |
| Any other branch | Patch (1.1.0 → 1.1.1) | `v1.1.1` |

### Pre-release Mode (`enable-prerelease: true`)

Set `enable-prerelease: "true"` to version feature branches before they're merged:

```yaml
- uses: TheKathan/semantic-versioning@v1
  with:
    major-version: "1"
    enable-prerelease: "true"
```

This creates tags with suffixes:

| Branch Pattern | Version Format | Example |
|----------------|----------------|---------|
| `feature/*`, `feat/*` | `v{major}.{minor}.{patch}-alpha.{n}` | `v1.2.0-alpha.1` |
| `fix/*`, `hotfix/*`, `bugfix/*` | `v{major}.{minor}.{patch}-fix.{n}` | `v1.1.1-fix.1` |
| `chore/*` | `v{major}.{minor}.{patch}-chore.{n}` | `v1.1.1-chore.1` |
| `docs/*` | `v{major}.{minor}.{patch}-docs.{n}` | `v1.1.1-docs.1` |
| `refactor/*` | `v{major}.{minor}.{patch}-refactor.{n}` | `v1.1.1-refactor.1` |
| `perf/*`, `performance/*` | `v{major}.{minor}.{patch}-perf.{n}` | `v1.1.1-perf.1` |
| `test/*`, `tests/*` | `v{major}.{minor}.{patch}-test.{n}` | `v1.1.1-test.1` |
| `build/*` | `v{major}.{minor}.{patch}-build.{n}` | `v1.1.1-build.1` |
| `ci/*` | `v{major}.{minor}.{patch}-ci.{n}` | `v1.1.1-ci.1` |
| `style/*` | `v{major}.{minor}.{patch}-style.{n}` | `v1.1.1-style.1` |

When you merge to main, it drops the suffix and creates the final version.

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `major-version` | `1` | Major version number |
| `enable-prerelease` | `false` | Enable pre-release tags on feature branches |
| `github-token` | `${{ github.token }}` | Token for API access |
| `branch-patterns` | `""` | Custom branch patterns (JSON format, optional) |
| `comment-on-pr` | `false` | Add version comment to PRs (requires `enable-prerelease: true`) |
| `add-to-summary` | `false` | Add version to GitHub Actions job summary |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The version tag that was created (e.g., `v1.2.3`) |
| `is-prerelease` | `true` if it's a pre-release, `false` otherwise |
| `branch` | Branch name used for versioning |
| `previous-version` | Previous version tag |
| `bump-type` | Type of version bump (Minor or Patch) |

## PR Comments & Job Summaries

Enable helpful version information directly in your pull requests and workflow summaries:

```yaml
- uses: TheKathan/semantic-versioning@v1
  with:
    major-version: "1"
    enable-prerelease: "true"
    comment-on-pr: "true"      # Adds comment to PRs
    add-to-summary: "true"      # Adds to workflow summary
```

Both features display the same information in a clean table format:

```
## 📦 Version: v1.2.0-alpha.1

| Field | Value |
|-------|-------|
| Type | Pre-release |
| Branch | feature/new-feature |
| Bump | Minor |
```

**Notes:**
- PR comments only appear when `enable-prerelease` is `true` (versions are created on feature branches)
- Job summaries appear on main branch merges or when `enable-prerelease` is `true`
- PR comments are updated in place (no duplicate comments)

## Creating Releases

If you want GitHub releases instead of just tags:

```yaml
- name: Version
  id: version
  uses: TheKathan/semantic-versioning@v1

- name: Create Release
  if: steps.version.outputs.is-prerelease == 'false'
  env:
    GH_TOKEN: ${{ github.token }}
  run: gh release create ${{ steps.version.outputs.version }} --generate-notes
```

## License

GPL-3.0

## Contributing

Found a bug or have an idea? Open an issue or submit a PR at [TheKathan/semantic-versioning](https://github.com/TheKathan/semantic-versioning)
