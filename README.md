# Semantic Versioning Action

[![Test and Release](https://github.com/TheKathan/semantic-versioning/actions/workflows/test.yml/badge.svg)](https://github.com/TheKathan/semantic-versioning/actions/workflows/test.yml)
[![version](https://img.shields.io/github/v/release/TheKathan/semantic-versioning?label=version)](https://github.com/TheKathan/semantic-versioning/releases/latest)

Automatically version your releases based on branch naming conventions. Works on main branch merges with optional pre-release support.

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

The action looks at your merged PR's source branch:
- `feature/*` or `feat/*` → bumps minor version (1.0.0 → 1.1.0)
- `fix/*`, `hotfix/*`, `chore/*` → bumps patch version (1.0.0 → 1.0.1)

It won't create duplicate tags if the commit already has one, and you can set your own major version.

## Pre-release Versions

Set `enable-prerelease: "true"` to version feature branches before they're merged:

```yaml
- uses: TheKathan/semantic-versioning@v1
  with:
    major-version: "1"
    enable-prerelease: "true"
```

This creates tags like:
- `feature/auth` → `v1.1.0-alpha.1`, `v1.1.0-alpha.2`, ...
- `fix/bug` → `v1.0.1-fix.1`, `v1.0.1-fix.2`, ...
- `chore/deps` → `v1.0.1-chore.1`, ...

When you merge to main, it drops the suffix and creates the final version.

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `major-version` | `1` | Major version number |
| `enable-prerelease` | `false` | Enable pre-release tags on feature branches |
| `github-token` | `${{ github.token }}` | Token for API access |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The version tag that was created (e.g., `v1.2.3`) |
| `is-prerelease` | `true` if it's a pre-release, `false` otherwise |

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
