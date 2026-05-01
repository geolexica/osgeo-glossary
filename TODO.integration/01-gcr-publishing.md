# 01 — Versioned GCR Publishing via glossarist Ruby Gem

## Goal

This repository publishes versioned GCR packages as GitHub Releases, triggered by version tags.

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

| Trigger | Tag | Assets |
|---------|-----|--------|
| Push tag `v1.2.0` | `v1.2.0` | `osgeo-1.2.0.gcr` + `osgeo.gcr` |
| workflow_dispatch | `v{version}` | `osgeo-{version}.gcr` + `osgeo.gcr` |

### Download URLs
```
https://github.com/geolexica/osgeo-glossary/releases/latest/download/osgeo.gcr
https://github.com/geolexica/osgeo-glossary/releases/download/v1.2.0/osgeo-1.2.0.gcr
```

## How to publish

```bash
git tag v1.2.0
git push origin v1.2.0
```

## Acceptance Criteria

- [ ] `publish-gcr.yml` triggers on `v*` tag push only
- [ ] `glossarist package` handles `geolexica-v2/` format
- [ ] Release has both versioned and unversioned GCR assets
- [ ] `metadata.yaml` has `shortname: osgeo` and `version`
- [ ] No checkout of vocabulary-browser
