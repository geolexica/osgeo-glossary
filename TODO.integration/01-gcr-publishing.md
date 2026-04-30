# 01 — GCR Package Publishing via glossarist Ruby Gem

## Goal

This repository publishes a GCR package as a GitHub Release asset on every push to `main`. The vocabulary-browser downloads this GCR to build www.geolexica.org.

## Repository Info

- **Dataset ID:** `osgeo`
- **Owner:** OSGeo
- **Concept format:** v2 (`geolexica-v2/*.yaml` — UUID-named multi-document YAML)
- **Concept count:** 444
- **Languages:** eng only
- **GCR release asset:** `osgeo.gcr`

## Current State

- `.github/workflows/publish-gcr.yml` checks out vocabulary-browser and uses its Node.js script
- Concepts are in `geolexica-v2/` directory (UUID YAML format) — NOT in `concepts/`
- The `concepts/` directory does NOT exist in git HEAD (was removed)
- Same v2 format as isotc211-glossary

## Key Challenge: v2 Format

Same as isotc211. Each file in `geolexica-v2/` is a multi-document YAML with a concept registry entry followed by localized concepts.

## Tasks

### 1. Verify glossarist Ruby gem handles v2 format

```bash
gem install glossarist

glossarist package /path/to/osgeo-glossary -o osgeo.gcr \
  --title "OSGeo Lexicon" \
  --owner "OSGeo"

glossarist validate osgeo.gcr
```

### 2. Replace publish-gcr.yml

```yaml
name: publish-gcr

on:
  push:
    branches: [main]
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

      - name: Install glossarist
        run: gem install glossarist

      - name: Build GCR package
        run: |
          glossarist package . -o osgeo.gcr \
            --title "OSGeo Lexicon" \
            --owner "OSGeo"

      - name: Update gcr-latest release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: gcr-latest
          name: "GCR Package (latest)"
          body: "Auto-generated GCR package. Updated on push to main."
          files: osgeo.gcr
```

### 3. Verify GCR content

```bash
glossarist validate osgeo.gcr
# Should show 444 concepts, 1 language (eng)
```

## GCR Download URL

```
https://github.com/geolexica/osgeo-glossary/releases/download/gcr-latest/osgeo.gcr
```

## Acceptance Criteria

- [ ] `publish-gcr.yml` uses `gem install glossarist` (no Node.js)
- [ ] `glossarist package` handles `geolexica-v2/` format
- [ ] GCR contains 444 concepts
- [ ] GCR uploaded to `gcr-latest` release
