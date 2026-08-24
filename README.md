# Game CI Templates

Reusable GitHub Actions workflows for building Unity game projects, so each project calls one
line instead of re-solving the same GameCI issues (like the disk-space failure below) from
scratch every time.

## `unity-build.yml`

Wraps [game-ci/unity-builder](https://github.com/game-ci/unity-builder) with two fixes baked
in from real GameCI runs on GitHub-hosted runners:

- **Disk space**: GitHub-hosted runners can run out of space pulling the Unity Docker image for
  a platform build. A `jlumbroso/free-disk-space` step runs first to strip unused preinstalled
  toolchains.
- **Library caching**: caches the `Library` folder keyed on `Assets`/`Packages`/`ProjectSettings`
  hashes, so unchanged projects don't re-import from scratch every run.

### Usage

In the calling project's own workflow file (e.g. `.github/workflows/build-android.yml`):

```yaml
name: Build Android

on:
  push:
    branches: [main]
  workflow_dispatch: {}

jobs:
  build:
    uses: splindeman/game-ci-templates/.github/workflows/unity-build.yml@main
    with:
      target-platform: Android
    secrets: inherit
```

### Inputs

| Input | Default | Description |
|---|---|---|
| `target-platform` | `Android` | GameCI `targetPlatform` — `Android`, `StandaloneWindows64`, `iOS`, etc. |
| `unity-version` | `auto` | Unity version, or `auto` to read it from the calling project |
| `artifact-name` | `build` | Prefix for the uploaded artifact name |
| `build-path` | `build` | Output directory GameCI writes into |

### Secrets

Requires `UNITY_LICENSE`, `UNITY_EMAIL`, `UNITY_PASSWORD` in the calling repo (Unity Hub →
Preferences → Licenses → Add → "Get a free personal license" to generate these) — pass them
through with `secrets: inherit`, or list them individually if being more selective.

### Cross-repo access note

This repo is currently **private**. GitHub's cross-repo reusable-workflow access for
user-owned (non-org) private repos needs to actually be verified end-to-end — see
`D:\Dev\tools\NOTES.md` for the outcome of the first real test (calling this from
three-spin-gauntlet). If it doesn't resolve cleanly, the fix is almost certainly making this
repo public — it holds no game-specific IP, just generic CI config.
