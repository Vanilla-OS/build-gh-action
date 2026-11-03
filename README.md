# Vanilla OS Desktop Image CI

Composite GitHub Action that builds a [Vanilla OS](https://vanillaos.org) desktop image from a [Vib](https://github.com/Vanilla-OS/Vib) recipe for the runner's architecture and uploads the resulting digest so a separate merge action can assemble the multi-arch manifest list.

## Quick start

```yaml
name: Vib Build

on:
  push:
    branches:
      - 'dev'
    tags:
      - '*'
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

      - uses: vanilla-os/build-gh-action@v1
        with:
          image: ghcr.io/${{ github.repository_owner }}/gnome
          base-image: ghcr.io/Vanilla-OS/core

  merge:
    name: Merge
    needs: build
    if: github.event_name != 'pull_request'
    runs-on: ubuntu-24.04
    permissions:
      contents: read
      packages: write
      id-token: write
      attestations: write
    steps:
      - uses: vanilla-os/merge-gh-action@v1
        with:
          image: ghcr.io/${{ github.repository_owner }}/gnome
```

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `recipe` | | `recipe.yml` | Path to the vib recipe. |
| `image` | ✅ | — | Target image name **without** a tag, e.g. `ghcr.io/owner/my-image`. Lowercased per the OCI spec. |
| `base-image` | | `ghcr.io/vanilla-os/core:latest` | Base image reference in `recipe.yml` that is verified/substituted. |
| `registry-username` | | `github.repository_owner` | Registry username. Only needed for non-GHCR registries. |
| `registry-password` | | `github.token` | Registry password or token. Only needed for non-GHCR registries. |

## Outputs

| Output | Description |
| --- | --- |
| `digest` | Digest of the pushed image. |
| `image-url` | Target image URL without a tag. |

The image is pushed on every non-`pull_request` event, and its digest is uploaded as a schema-free marker file named `digests-<arch>` (one empty file per architecture, named after the digest).