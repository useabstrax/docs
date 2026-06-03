# Contributing

- [Building from source](/docs/contributing/building-from-source)

This page is for contributors and maintainers. It describes how the project is organised, how to set up a development environment, and how the project is built, tested, and released.

Abstrax is a single Go binary. There is no web frontend, no database, and no services to run during development — development is building and testing a Go binary.

## Requirements

- Go 1.22 or newer (the module declares `go 1.22`).
- A Linux environment for running most commands. The binary builds on any platform Go supports, but the commands themselves target Debian/Ubuntu.

## Repository layout

The Go module lives in the `cli/` directory.

```text
cli/
  cmd/abstrax/            Program entry point (main.go)
  internal/
    cli/                  Cobra command definitions, one file per command group
    actions/              Stable action name constants used in output and the future agent
    backup/               Timestamped file/directory backup helpers
    confirm/              Interactive yes/no confirmation prompts
    exec/                 Wrapper around os/exec with dry-run and verbose support
    globals/              Parsed global flag values
    output/               Human-readable and JSON output helpers
    platform/             OS detection and Debian/Ubuntu constants
      debian/             Paths and constants for Debian/Ubuntu
    services/             One package per area, holding the real logic
      user/ sshkey/ sshcfg/ pkgmanager/ svcmanager/ cron/ daemon/
      project/ web/ ssl/ mysql/ cache/ firewall/ serverinfo/
    validate/             Input validation helpers
    version/              Version variables set at build time
  packaging/              systemd unit and package post-install script
  .github/workflows/      CI (test) and release workflows
  .goreleaser.yaml        Release build configuration
```

## Architecture

The flow for every command is the same:

```text
CLI command (internal/cli)
  -> validates input (internal/validate)
  -> builds a typed options struct
  -> calls a service (internal/services/<area>)
  -> service uses platform adapters and the exec runner
  -> returns a structured result
  -> CLI prints human output or JSON (internal/output)
```

Key conventions:

- **Commands** live in `internal/cli`, one file per group (`user.go`, `firewall.go`, and so on). They are built with [Cobra](https://github.com/spf13/cobra).
- **Services** in `internal/services/<area>` hold the real logic and are kept separate from the CLI layer. This separation is intentional so the planned hosted agent can reuse the same logic.
- **Action names** in `internal/actions` are stable identifiers (for example `user.add`). They appear in JSON output and are reserved for the future agent. Treat them as a stable API.
- **Output** always goes through `internal/output`, which renders either human text or JSON based on the `--json` flag.
- **Global flags** are read from `internal/globals`, not threaded through every function.
- **Command execution** goes through `internal/exec`, which supports `--dry-run` (printing the command instead of running it) and `--verbose` (printing the command before running it).

## Install dependencies

From the `cli/` directory:

```bash
go mod download
```

## Build

```bash
go build -o abstrax ./cmd/abstrax
```

The resulting `./abstrax` binary sits in the repo and is `.gitignore`d. See [Building from source](/docs/contributing/building-from-source) for more, including version metadata.

## Run

```bash
./abstrax --help
./abstrax doctor
```

`doctor` works without root and is a good smoke test after building. Most other commands require root; run them with `sudo ./abstrax ...`. Use `--dry-run` to preview behaviour without making changes.

## Tests

Run the full test suite with the race detector:

```bash
go test -v -race ./...
```

There are unit tests for the validation helpers (`internal/validate`) and platform detection (`internal/platform`). When you add behaviour, add tests alongside it.

## Formatting and vetting

The CI checks formatting and runs `go vet`. Match it locally before opening a pull request:

```bash
# Formatting: no output means clean
gofmt -l .

# Static checks
go vet ./...
```

If `gofmt -l .` lists files, format them:

```bash
gofmt -w .
```

## Continuous integration

The test workflow (`.github/workflows/test.yml`) runs on pushes to `main` and on pull requests. It performs, in order:

1. `gofmt -l .` — fails if any file is not formatted.
2. `go vet ./...`
3. `go test -v -race ./...`
4. `go build -o abstrax ./cmd/abstrax`

Make sure all four pass locally before submitting changes.

## Releases

Releases are built with [GoReleaser](https://goreleaser.com/) via `.github/workflows/release.yml`:

- Pushes to `main` (without a tag) build `linux/amd64` and `linux/arm64` binaries and upload them as workflow artifacts.
- Pushing a tag matching `v*` runs GoReleaser, which builds the binaries, produces `tar.gz` archives and `.deb`/`.rpm` packages, generates checksums, and creates a GitHub release.

See [Building from source](/docs/contributing/building-from-source#release-builds) for the GoReleaser configuration details.

## Contributing checklist

1. Fork the repository and create a feature branch.
2. Make your change, with tests where appropriate.
3. Ensure `gofmt -l .` is clean.
4. Ensure `go vet ./...` passes.
5. Ensure `go test ./...` passes.
6. Open a pull request.

## Related

- [Building from source](/docs/contributing/building-from-source)
- [Command reference](/docs/reference/index)
