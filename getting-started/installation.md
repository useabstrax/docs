# Installation

- [Installation](/getting-started/installation.html)
- [Updating](/getting-started/updating.html)
- [Uninstalling](/getting-started/uninstalling.html)

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

See [Supported platforms](/reference/supported-platforms.html) for more detail.

## Option 1: Release archive

Releases are published as compressed archives on the [releases page](https://github.com/useabstrax/agent/releases). The release builds produce `tar.gz` archives for `linux/amd64` and `linux/arm64`. The archive name includes the version, for example `abstrax_1.0.0_linux_amd64.tar.gz`.

Download the archive for your architecture (check with `uname -m`), extract the `abstrax` binary, and move it onto your `PATH`:

```bash
# Replace <version> with the release version, for example 1.0.0
tar -xzf abstrax_<version>_linux_amd64.tar.gz
chmod +x abstrax
sudo mv abstrax /usr/local/bin/abstrax
```

Each release also publishes a checksums file (`abstrax_<version>_checksums.txt`) using SHA-256. You can verify your download against it:

```bash
sha256sum -c abstrax_<version>_checksums.txt 2>&1 | grep abstrax_<version>_linux_amd64.tar.gz
```

## Option 2: Debian or RPM package

Release builds also produce `.deb` and `.rpm` packages, named `abstrax_<version>_<arch>` (for example `abstrax_1.0.0_amd64.deb`). Installing the `.deb` places the binary at `/usr/bin/abstrax` and creates the standard Abstrax directories.

```bash
# Replace <version> and <arch> with the values from the release
sudo dpkg -i abstrax_<version>_<arch>.deb
```

The package post-install step creates these directories with restricted permissions:

```text
/etc/abstrax        (0750)
/var/lib/abstrax    (0750)
/var/log/abstrax    (0750)
```

It also installs a systemd unit file for a future agent at `/etc/systemd/system/abstrax-agent.service`. This service is intentionally **not** enabled or started, because the agent is not yet implemented.

## Option 3: Build from source

See [Building from source](/contributing/building-from-source.html) for full detail. In short:

```bash
git clone https://github.com/useabstrax/agent
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

- [Quick start](/getting-started/index.html)
- [Updating](/getting-started/updating.html)
- [Uninstalling](/getting-started/uninstalling.html)
