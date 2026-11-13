# Contributing to the Sonarr & Radarr Autoscan Plugin

The [Silo contribution guide](https://github.com/Silo-Server/.github/blob/main/CONTRIBUTING.md)
covers project-wide coordination, focused changes, evidence, AI disclosure, and
pull request expectations. Those requirements apply here; this guide adds the
plugin-specific workflow.

## Before you start

Open an [issue](https://github.com/Silo-Server/silo-plugin-autoscan-arr/issues)
before changing polling semantics, markers, path handling, configuration, or
the advertised capability. This repository owns the Sonarr/Radarr adapter;
contract changes belong in
[`silo-plugin-sdk`](https://github.com/Silo-Server/silo-plugin-sdk), while host
scheduling and scan behavior belong in
[`silo-server`](https://github.com/Silo-Server/silo-server).

## Development setup

Use the Go version declared in `go.mod`. A local `go.work` may point at a sibling
SDK checkout while developing both repositories, but committed code and CI must
resolve the tagged SDK dependency with `GOWORK=off`. Never commit a local
filesystem `replace` directive.

## Validate your change

```sh
GOWORK=off go test ./...
GOWORK=off go test -tags integration ./...
GOWORK=off go vet ./...
GOWORK=off go build ./...
gofmt -l .
```

`gofmt -l .` should print nothing. If it reports unrelated pre-existing drift,
none of the Go files touched by your change may appear in the output; do not add
to the output, and report what remains. Add focused coverage for cursor
ordering, history pagination, event filtering, credential handling, and path
extraction when those behaviors change.

## Open the pull request

Use a Conventional Commit title, explain any compatibility or rescan risk, and
paste the actual validation results. Read the
[AI-assisted contribution policy](https://github.com/Silo-Server/silo-server/blob/main/docs/ai-contributions.md)
and include its disclosure block.

## Release

Maintainers cut releases. Pushing a `v<semver>` tag runs
[`release.yml`](.github/workflows/release.yml), which calls the shared
`Silo-Server/.github` plugin release workflow. It builds the binaries, signs
them with GitHub artifact attestations, publishes the GitHub release, and asks
the [`silo-plugins`](https://github.com/Silo-Server/silo-plugins) catalog to
pick it up. There is no manual-dispatch release.

1. Open a pull request that sets `version` in `manifest.json` to the new
   version (for example `0.1.3`), and merge it.
2. Tag the commit that pull request put on `main` and push the tag. Tag that
   commit itself, not a later one:

   ```sh
   git switch main && git pull --ff-only
   git tag v0.1.3 <commit that changed manifest.json>
   git push origin v0.1.3
   ```

The workflow fails without publishing anything if the tag is not `v<semver>`,
`manifest.json` `version` does not match the tag, or the tagged commit is not
the commit on `main` that changed `version`. Only that one commit can be
released for a version. Do not move or delete a pushed tag; fix `manifest.json`
in a new pull request and tag the next version.

Each release carries `plugin-<os>-<arch>` binaries, `checksums.txt`, and
`plugin.sigstore.json`. To verify a download, check its attestation with the
GitHub CLI, naming the release's tag:

```sh
gh attestation verify plugin-linux-amd64 \
  --repo Silo-Server/silo-plugin-autoscan-arr \
  --cert-identity https://github.com/Silo-Server/.github/.github/workflows/plugin-release.yml@refs/heads/main \
  --source-ref refs/tags/vX.Y.Z \
  --deny-self-hosted-runners
```

This is the check Silo servers apply: the shared release workflow on its
`main` branch built the binary from that tag of this repository. To check
against the bundle shipped with the release instead of fetching the attestation
from GitHub, add `--bundle plugin.sigstore.json`; `gh` still fetches Sigstore's
trusted root unless you follow GitHub's
[offline verification steps](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/verify-attestations-offline).
Verify before running a downloaded binary.
`sha256sum plugin-linux-amd64` must also match the binary's line in
`checksums.txt`.
