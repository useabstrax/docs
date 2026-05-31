# Building from source

This page covers building the Abstrax binary yourself, including how version information is embedded and how release packages are produced.

## Requirements

- Go 1.22 or newer.
- Git (to clone the repository and to embed commit metadata).

## Clone and build

The Go module is in the `cli/` directory.

```bash
git clone https://github.com/abstrax-io/abstrax
cd abstrax/cli
go mod download
go build -o abstrax ./cmd/abstrax
```

Verify the build:

```bash
./abstrax --help
./abstrax doctor
```

Install it onto your `PATH`:

```bash
sudo mv abstrax /usr/local/bin/abstrax
```

A plain `go build` produces a binary whose version reads:

```text
abstrax dev (commit none, built unknown)
```

This is expected for local builds. To embed real version metadata, see the next section.

## Embedding version metadata

The version, commit, and build date are package-level variables in `internal/version` that are set at build time with `-ldflags`. To embed them, mirror what the release workflow does:

```bash
go build \
  -ldflags="-s -w \
    -X abstrax/internal/version.Version=$(git describe --tags --always --dirty) \
    -X abstrax/internal/version.Commit=$(git rev-parse --short HEAD) \
    -X abstrax/internal/version.BuildDate=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  -o abstrax ./cmd/abstrax
```

Then:

```bash
./abstrax version
```

## Cross-compiling

Go cross-compiles without extra tooling. To build for a different architecture:

```bash
# Linux amd64
GOOS=linux GOARCH=amd64 go build -o dist/abstrax-linux-amd64 ./cmd/abstrax

# Linux arm64
GOOS=linux GOARCH=arm64 go build -o dist/abstrax-linux-arm64 ./cmd/abstrax
```

These are the two targets the release workflow builds.

## Running tests before building

It is good practice to run the test suite first:

```bash
go test ./...
go test -v -race ./...
```

## Release builds

Releases use [GoReleaser](https://goreleaser.com/), configured in `.goreleaser.yaml`. The configuration:

- Builds `linux/amd64` and `linux/arm64` with `CGO_ENABLED=0` and the version `-ldflags` shown above.
- Produces `tar.gz` archives that include `README.md` and the agent systemd unit.
- Produces `.deb` and `.rpm` packages that install the binary to `/usr/bin/abstrax`, create `/etc/abstrax`, `/var/lib/abstrax`, and `/var/log/abstrax` with mode 0750, and install the agent systemd unit at `/etc/systemd/system/abstrax-agent.service`.
- Runs a post-install script (`packaging/scripts/postinstall.sh`) that creates the directories, sets permissions, and prints a getting-started message. It does not enable the agent service, which is not implemented.
- Generates SHA-256 checksums.

### Build release artefacts locally

To produce the release artefacts on your machine without publishing them:

```bash
goreleaser release --snapshot --clean
```

The output is written to the `dist/` directory.

### How releases are triggered

The release workflow (`.github/workflows/release.yml`) runs:

- On pushes to `main` (without a tag): builds the two Linux binaries and uploads them as workflow artifacts.
- On pushing a tag matching `v*`: runs `goreleaser release --clean`, which builds everything and creates a GitHub release.

## Related

- [Development](development.md)
- [Installation](../getting-started/installation.md)
- [Supported platforms](../reference/supported-platforms.md)
