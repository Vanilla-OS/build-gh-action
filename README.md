# Vanilla OS Desktop Image CI

Composite GitHub Action that builds, pushes, and merges [Vanilla OS](https://vanillaos.org) desktop images from a [Vib](https://github.com/Vanilla-OS/Vib) recipe.

The action has two modes that are meant to run in the same workflow run:

- **`build`** — builds the recipe for the runner's architecture with Buildah, pushes it to the registry.
- **`merge`** — collects the per-architecture digests from all build jobs, creates a multi-arch manifest list, pushes it under the final tags, and attests it.

## Quick start

```yaml
name: Vib Build

on:
  push:
    branches: [ dev ]
    tags:
  pull_request:

permissions:
  contents: read
  packages: write
  id-token: write
  attestations: write

jobs:
  build:
    name: Build ${{ matrix.arch }}
    strategy:
      fail-fast: false
      matrix:
        include:
          - runner: ubuntu-24.04
          - runner: ubuntu-24.04-arm
    runs-on: ${{ matrix.runner }}
    steps:
      - uses: actions/checkout@v7

      - uses: vanilla-os/builder-gh-action@v1
        with:
          mode: build
          image: ghcr.io/${{ github.repository_owner }}/gnome

  merge:
    name: Merge manifest
    needs: build
    if: github.event_name != 'pull_request'
    runs-on: ubuntu-24.04
    steps:
      - uses: vanilla-os/builder-gh-action@v1
        with:
          mode: merge
          image: ghcr.io/${{ github.repository_owner }}/gnome
```

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `mode` | ✅ | — | `build` or `merge`. |
| `recipe` | | `recipe.yml` | Path to the vib recipe. |
| `image` | ✅ | — | Target image name **without** a tag, e.g. `ghcr.io/owner/my-image`. Lowercased per the OCI spec. |
| `tags` | | `latest` | Comma-separated tags. In `build` mode the **first** tag is reused as the `base-image` tag. |
| `legacy-image` | | — | Optional legacy image reference including a tag (e.g. `ghcr.io/owner/desktop:main`). When set, `merge` also pushes the manifest list there. |
| `base-image` | | `ghcr.io/vanilla-os/core:latest` | Base image reference in `recipe.yml` that is verified/substituted. |
| `registry-username` | | `github.repository_owner` | Registry username. |
| `registry-password` | | `github.token` | Registry password or token. |

## Outputs

| Output | Description |
| --- | --- |
| `digest` | Digest of the built image (`build` mode) or of the merged manifest list (`merge` mode). |
| `image-url` | Target image URL without a tag. |