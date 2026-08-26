# duckdb-extensions

Pinned DuckDB extension sets for [DuckFlight](https://github.com/fairtier/duckflight)
boxes, published as `ghcr.io/fairtier/duckdb-extensions`.

The image contains only `/extensions/<duckdb-version>/<platform>/*.duckdb_extension`
(on a busybox base) and is meant to run as the DuckFlight chart's
`extensions.image` initContainer: it copies `/extensions` into an emptyDir
mounted over the server's `EXTENSION_DIR`, giving a per-deployment extension
set without a per-deployment server image.

## How it builds

There is no Dockerfile here. The release workflow builds duckflight's own
Dockerfile via a **remote git context pinned to a duckflight tag**, targeting
its `extensions` stage, with the extension lists as build args:

```
docker buildx build \
  --target extensions \
  --build-arg CORE_EXTENSIONS="iceberg avro httpfs" \
  --build-arg COMMUNITY_EXTENSIONS="" \
  https://github.com/fairtier/duckflight.git#v0.2.0
```

## Version coupling — read before bumping anything

Extension binaries are tied to the **DuckDB version inside the duckflight
binary** (the Dockerfile derives it from the pinned `duckdb-go` module).
Therefore:

- the image tag is `<duckflight tag>-<set>.<n>` (e.g. `0.2.0-gsheets.1`), and
- a box must run this image **only next to the duckflight image built from
  the same tag** (`apps/box/duckflight/values.yaml` pins both).

Bumping duckflight means re-releasing this image against the new tag, even
when the extension list is unchanged.

## Supply chain

Extensions are fetched at build time from `extensions.duckdb.org` (core) and
`community-extensions.duckdb.org` (community, third-party; the set is empty
since query-time federation was retired 2026-08-27), then baked and pinned by
image digest. Boxes never
install extensions at runtime; DuckFlight refuses client-issued
`INSTALL`/`LOAD` (`REJECT_CLIENT_EXTENSIONS`).
