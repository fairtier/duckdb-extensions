# duckdb-extensions

**Extras** image for [DuckFlight](https://github.com/fairtier/duckflight)
boxes, published as `ghcr.io/fairtier/duckdb-extensions`.

The image contains only `/extensions/<duckdb-version>/<platform>/*.duckdb_extension`
(on a busybox base) and is meant as the DuckFlight chart's
`extensions.image`: extension binaries layered **on top of** the set baked
into the server image. Since chart `helm/v0.0.7` the mount is a union — the
chart seeds the baked set (iceberg, avro, httpfs) from the server image
itself and overlays this image second — so an extras image ships *only its
extras*, never the mandatory set.

**Currently dormant.** Query-time federation was retired 2026-08-27 (gsheets
left with it) and no box loads an extra engine extension; no image tag is
consumed anywhere. The repo stays as the door: when a box needs an extra
extension in the SQL engine again, fill the list in
`.github/workflows/release.yml` and push a tag. Pipeline (EL) extensions are
a different thing entirely — they bake into the dlt-worker image
(`docs/plans/duckdb-source.md` in the platform repo).

## How it builds

There is no Dockerfile here. The release workflow builds duckflight's own
Dockerfile via a **remote git context pinned to a duckflight tag**, targeting
its `extensions` stage, with the extension lists as build args:

```
docker buildx build \
  --target extensions \
  --build-arg CORE_EXTENSIONS="" \
  --build-arg COMMUNITY_EXTENSIONS="some_extra" \
  https://github.com/fairtier/duckflight.git#v0.2.1
```

## Version coupling — read before bumping anything

Extension binaries are tied to the **DuckDB version inside the duckflight
binary** (the Dockerfile derives it from the pinned `duckdb-go` module).
Therefore:

- the image tag is `<duckflight tag>-<set>.<n>` (e.g. `0.2.1-core.1`), and
- a box must run this image **only next to the duckflight image built from
  the same tag** (`apps/box/duckflight/values.yaml` pins both).

Bumping duckflight means re-releasing this image against the new tag, even
when the extension list is unchanged.

## Supply chain

Extensions are fetched at build time from `extensions.duckdb.org` (core) and
`community-extensions.duckdb.org` (community, third-party), then baked and
pinned by image digest. Boxes never install extensions at runtime; DuckFlight
refuses client-issued `INSTALL`/`LOAD` (`REJECT_CLIENT_EXTENSIONS`).
