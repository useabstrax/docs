# Installation

Abstrax is distributed as a single binary. There is no runtime to install and no background service to run. This page covers the installation options that are currently supported.

## System requirements

- A 64-bit Linux server (`amd64` or `arm64`).
- Root or `sudo` access for most commands that change system state.
- The underlying tools you intend to manage. Abstrax calls these tools rather than replacing them, so for example managing nginx requires nginx to be installed. Run `abstrax doctor` to see which tools are present.

To build from source you also need Go 1.22 or newer.

## Supported operating systems

Abstrax targets Debian and Ubuntu based systems. The platform detection in the code treats the following as supported:

- Ubuntu
- Debian
- Linux Mint
- Pop!_OS
- Raspbian

On any other distribution Abstrax reports the platform as not fully supported. Detection is based on the `ID` field in `/etc/os-release`.

The package management commands use `apt`, and many service operations assume `systemd`, so the most reliable experience is on current Ubuntu and Debian releases.

See [Supported platforms](/docs/reference/supported-platforms) for more detail.

## Option 1: Install from a release archive (recommended)

This is the recommended and most transparent way to install Abstrax. Every step is visible, so you can see exactly what is downloaded and verify it before it runs. This method:

- downloads the release archive directly from the official [GitHub releases page](https://github.com/useabstrax/abstrax/releases)
- verifies the archive using the published SHA-256 checksums file
- extracts the `abstrax` binary
- moves it onto your `PATH`

Releases are published as compressed archives on the [releases page](https://github.com/useabstrax/abstrax/releases). GoReleaser builds `tar.gz` archives for `linux/amd64` and `linux/arm64`, named `abstrax_<version>_linux_<arch>.tar.gz` (for example `abstrax_1.0.0_linux_amd64.tar.gz`). Each release tag uses a `v` prefix (for example `v1.0.0`), but the version in the filename omits it.

Set the release version and your architecture, then download, verify, and install:

```bash
# Replace with the release version from the releases page, without the leading v
VERSION="1.0.0"

# x86_64 servers use amd64; aarch64/ARM servers use arm64 (check with: uname -m)
ARCH="amd64"

wget "https://github.com/useabstrax/abstrax/releases/download/v${VERSION}/abstrax_${VERSION}_linux_${ARCH}.tar.gz"
wget "https://github.com/useabstrax/abstrax/releases/download/v${VERSION}/abstrax_${VERSION}_checksums.txt"

sha256sum -c "abstrax_${VERSION}_checksums.txt" 2>&1 | grep "abstrax_${VERSION}_linux_${ARCH}.tar.gz"

tar -xzf "abstrax_${VERSION}_linux_${ARCH}.tar.gz"
chmod +x abstrax
sudo mv abstrax /usr/local/bin/abstrax
```

To install the latest release without looking up the version number:

```bash
VERSION=$(wget -qO- "https://api.github.com/repos/useabstrax/abstrax/releases/latest" | grep -Po '"tag_name": "\K[^"]+' | sed 's/^v//')
ARCH="amd64"   # or arm64

wget "https://github.com/useabstrax/abstrax/releases/download/v${VERSION}/abstrax_${VERSION}_linux_${ARCH}.tar.gz"
wget "https://github.com/useabstrax/abstrax/releases/download/v${VERSION}/abstrax_${VERSION}_checksums.txt"

sha256sum -c "abstrax_${VERSION}_checksums.txt" 2>&1 | grep "abstrax_${VERSION}_linux_${ARCH}.tar.gz"

tar -xzf "abstrax_${VERSION}_linux_${ARCH}.tar.gz"
chmod +x abstrax
sudo mv abstrax /usr/local/bin/abstrax
```

Each release also publishes a SHA-256 checksums file named `abstrax_<version>_checksums.txt`. The commands above download it and verify the archive before extracting.

## Option 2: Install using the convenience script

The convenience script performs the same basic steps as the manual release archive method above. It:

- detects the server architecture
- finds the latest GitHub release
- downloads the correct archive from GitHub
- downloads the checksums file
- verifies the archive with SHA-256
- extracts the binary
- installs it to `/usr/local/bin/abstrax`
- prints the installed version
- suggests running `abstrax doctor`

If you would like to read the script before running it, download and inspect it first:

```bash
wget https://useabstrax.com/install.sh
less install.sh
sudo bash install.sh
```

Or install directly in a single command:

```bash
wget -qO- https://useabstrax.com/install.sh | sudo bash
```

If you would prefer not to pipe a script into `sudo bash`, use the manual release archive instructions above. They produce the same result with every step visible.

## Option 3: Build from source

See [Building from source](/docs/contributing/building-from-source) for full detail. In short:

```bash
git clone https://github.com/useabstrax/abstrax
cd abstrax/cli
go mod download
go build -o abstrax ./cmd/abstrax
sudo mv abstrax /usr/local/bin/abstrax
```

## Verify the installation

Check that the binary is on your `PATH` and runs:

```bash
abstrax --help
```

Then run the system inspection command, which works without root and is a good first check:

```bash
abstrax doctor
```

If `doctor` reports your OS, package manager, and service manager correctly, the installation is working.

## Check the installed version

```bash
abstrax version
```

This prints the version, commit, and build date, for example:

```text
abstrax 1.0.0 (commit abc1234, built 2024-01-01T12:00:00Z)
```

For a binary built locally without release metadata, the version reads `dev (commit none, built unknown)`.

You can also get this as JSON:

```bash
abstrax version --json
```

```json
{
  "status": "success",
  "action": "version.show",
  "summary": "abstrax 1.0.0 (commit abc1234, built 2024-01-01T12:00:00Z)",
  "data": {
    "version": "1.0.0",
    "commit": "abc1234",
    "build_date": "2024-01-01T12:00:00Z"
  }
}
```

## Next steps

- [Quick start](/docs/getting-started/quick-start)
- [Updating](/docs/getting-started/updating)
- [Uninstalling](/docs/getting-started/uninstalling)
