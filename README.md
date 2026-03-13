# GH-shared-workflows

Reusable GitHub Actions workflows for common CI/CD tasks.

## Available Workflows

- [`laravel-zero-build.yml`](#laravel-zero-buildyml) — Build Laravel Zero PHAR archives and standalone binaries, publish to a bin branch, and create GitHub releases.

---

## `laravel-zero-build.yml`

Builds a Laravel Zero application into a PHAR archive, with optional standalone binary compilation via [phpacker](https://github.com/nicoder-php/phpacker). In release mode, it publishes the PHAR to a dedicated bin branch and creates a GitHub release with all build artifacts.

### Workflow Jobs

1. **prepare** — Validates inputs and resolves the build version (git SHA for snapshots, explicit version for releases). Parses phpacker target platforms into a matrix.
2. **build-phar** — Installs PHP and Composer dependencies, builds the PHAR using `app:build`.
3. **build-binaries** *(optional)* — Uses phpacker to compile standalone binaries for each target platform from the built PHAR. Only runs when `build-binaries: true`.
4. **publish** *(release only)* — Commits the PHAR to the bin branch and tags it.
5. **release** *(release only)* — Creates a GitHub release on the source tag with the PHAR and zipped binaries attached.

### Usage

```yaml
name: Build and Release

on:
  push:
    tags:
      - '*-src'

jobs:
  build:
    uses: kduma-OSS/GH-shared-workflows/.github/workflows/laravel-zero-build.yml@main
    with:
      php-version: '8.4'
      binary-name: 'my-app'
      release: true
      version: ${{ github.ref_name }}
      src-tag-name: ${{ github.ref_name }}
      bin-tag-name: ${{ github.ref_name }}-bin
      build-binaries: true
      phpacker-targets: 'linux, mac, mac-arm, win'
    permissions:
      contents: write
```

> For a real-world example, see the [CLI-php-ca release workflow](https://github.com/kduma-OSS/CLI-php-ca/blob/develop/.github/workflows/release.yml).

### Inputs

| Input | Type | Required | Default | Description |
|---|---|---|---|---|
| `php-version` | string | **yes** | — | PHP version to use |
| `binary-name` | string | **yes** | — | Name of the binary file |
| `release` | boolean | no | `false` | Release mode (true) or snapshot mode (false) |
| `version` | string | no | `''` | Version string for the build |
| `src-tag-name` | string | no | `''` | Tag name for GitHub release (e.g. `v1.0.0-src`) |
| `bin-tag-name` | string | no | `''` | Tag name on bin branch (e.g. `v1.0.0`) |
| `version-prefix` | string | no | `'v'` | Prefix for version in messages |
| `bin-branch` | string | no | `'bin'` | Branch name for binary releases |
| `generate-release-notes` | boolean | no | `true` | Generate release notes automatically |
| `retention-days` | number | no | `7` | Artifact retention days for snapshot builds |
| `artifact-name` | string | no | `'binary'` | Name for the uploaded build artifact |
| `runs-on` | string | no | `'ubuntu-latest'` | Runner to use |
| `install-dev-deps` | boolean | no | `false` | Install dev dependencies (omit `--no-dev`) |
| `build-binaries` | boolean | no | `false` | Build standalone binaries using phpacker |
| `phpacker-targets` | string | no | `'linux, linux-arm, mac, mac-arm, win'` | phpacker platform argument (available: `linux`, `linux-arm`, `mac`, `mac-arm`, `win`) |

### Permissions

The caller workflow must grant `contents: write` permission when using release mode, as the workflow needs to push to the bin branch and create GitHub releases.

```yaml
permissions:
  contents: write
```
