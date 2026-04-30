# 01 — Versioned GCR Publishing via glossarist Ruby Gem

## Goal

This repository publishes versioned GCR packages as GitHub Release assets.

## Repository Info

| Field | Value |
|-------|-------|
| **Dataset ID / shortname** | `osgeo` |
| **Owner** | OSGeo |
| **Format** | v2 (`geolexica-v2/*.yaml` — UUID multi-document YAML) |
| **Concepts** | 444 |
| **Languages** | eng only |
| **GCR filename** | `osgeo-{version}.gcr` |

## Release Convention

| Trigger | Tag | Asset |
|---------|-----|-------|
| Push to `main` | `gcr-latest` (rolling) | `osgeo.gcr` |
| Tag `gcr-v1.2.0` | `gcr-v1.2.0` (pinned) | `osgeo-1.2.0.gcr` |

## Tasks

### 1. Replace publish-gcr.yml with glossarist gem

```yaml
name: publish-gcr

on:
  push:
    branches: [main]
    tags: ['gcr-v*']
  workflow_dispatch:

permissions:
  contents: write

jobs:
  publish-gcr:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'

      - run: gem install glossarist

      - name: Determine version
        id: version
        run: |
          if [[ "${GITHUB_REF}" == refs/tags/gcr-v* ]]; then
            VERSION="${GITHUB_REF_NAME#gcr-v}"
            echo "version=${VERSION}" >> "$GITHUB_OUTPUT"
            echo "tag=gcr-v${VERSION}" >> "$GITHUB_OUTPUT"
            echo "filename=osgeo-${VERSION}.gcr" >> "$GITHUB_OUTPUT"
          else
            DATEVER=$(date +%Y.%m.%d)
            echo "version=${DATEVER}" >> "$GITHUB_OUTPUT"
            echo "tag=gcr-latest" >> "$GITHUB_OUTPUT"
            echo "filename=osgeo.gcr" >> "$GITHUB_OUTPUT"
          fi

      - name: Build GCR package
        run: |
          glossarist package . -o "${{ steps.version.outputs.filename }}" \
            --shortname osgeo \
            --version "${{ steps.version.outputs.version }}" \
            --title "OSGeo Lexicon" \
            --owner "OSGeo"

      - name: Publish release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ steps.version.outputs.tag }}
          name: "GCR Package ${{ steps.version.outputs.version }}"
          files: ${{ steps.version.outputs.filename }}
```

### 2. Verify locally

```bash
gem install glossarist
glossarist package . -o osgeo-test.gcr \
  --shortname osgeo --version 0.0.1 \
  --title "OSGeo Lexicon" --owner "OSGeo"
glossarist validate osgeo-test.gcr
```

### 3. Remove vocabulary-browser dependency

## Acceptance Criteria

- [ ] `publish-gcr.yml` uses `gem install glossarist` only
- [ ] GCR contains 444 concepts
- [ ] `metadata.yaml` has `shortname: osgeo` and `version`
- [ ] No checkout of vocabulary-browser
