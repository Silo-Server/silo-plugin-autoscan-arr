# silo-plugin-autoscan-arr

A Silo plugin that implements the **`scan_source.v1`** capability for **Sonarr / Radarr**.
When Silo's host polls it, the plugin reads the arr instance's recent history,
extracts imported and renamed file paths, and hands the **raw arr-side paths** back to
the host. The host applies any configured path rewrites and triggers targeted library
rescans.

## How it works

Silo's host owns the polling timer and calls `PollChanges(marker, connection)` on
each tick. The plugin:

1. Reads the resolved arr `{base_url, api_key}` from `PollChangesRequest.connection`
   (the host resolves the operator's connection — own credentials or a reused
   Requests link — and passes them per call). **The plugin stores no credentials.**
2. Queries arr `/api/v3/history` (paginated: `page` / `pageSize` /
   `sortKey=date` / `sortDirection=descending`), bounded to a 24h max-lookback
   with a 1-minute overlap so boundary events aren't missed and a long-idle
   source doesn't replay full history.
3. Extracts `downloadFolderImported` (new files) and `episodeFileRenamed` /
   `movieFileRenamed` (both the new and old path). Deletes are ignored — upgrade
   deletes are covered by the paired import.
4. Returns the **raw arr-side paths** in `PollChangesResponse.source_paths` plus an
   opaque `next_marker` (a composite `<RFC3339>|<id>` cursor that never regresses
   below the caller's marker; a bare RFC3339 string is still accepted for backward
   compatibility). **Path rewrites are applied by the host**, not the plugin.

## Configuration

The plugin has **no configuration**. `global_config_schema` is empty (`[]`).

The arr connection (URL + API key) is supplied by the host on every poll via
`PollChangesRequest.connection` — it is never stored by the plugin. Path rewrites are
configured in the host and applied after the plugin returns source paths.

## Building

```sh
make build          # build the binary for the current platform
make build-all      # cross-platform binaries (dist/)
go test ./...       # unit tests
go test -tags integration ./...   # spawns the real binary over go-plugin gRPC
```

## Status

Depends on `silo-plugin-sdk` ≥ the release that adds `scan_source.v1` with the
`connection` field on `PollChangesRequest` and `source_paths` on `PollChangesResponse`.
`go.mod` depends on the SDK via a pseudo-version (no local `replace`); bump it to a
tagged release once one is published. Catalog registration in `silo-plugins` is a
separate step.
