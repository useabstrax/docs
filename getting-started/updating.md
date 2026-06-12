# Updating

Abstrax can update itself from GitHub releases. You can use the built-in `self update` command (recommended), follow the manual release archive steps (the same method used during [installation](/docs/getting-started/)), or rebuild from source if you installed that way.

Updating the binary does not change anything you have already configured on the server. Users, firewall rules, projects, cron jobs, databases, and certificates live in the system, not in the binary. Abstrax state files in `/var/lib/abstrax` and config in `/etc/abstrax` are not modified by an update.

## Check your current version

```bash
abstrax version
```

This prints the version, commit, and build date, for example:

```text
abstrax 1.0.1 (commit abc1234, built 2024-01-01T12:00:00Z)
```

For JSON output:

```bash
abstrax version --json
```

## Option 1: Self-update (recommended)

`abstrax self update` downloads the correct release archive from GitHub, verifies it using the published SHA-256 checksums file, extracts the binary, and replaces the currently running executable. It requires root and works on `linux/amd64` and `linux/arm64`.

Update to the newest compatible release:

```bash
sudo abstrax self update
```

By default, Abstrax follows [semantic versioning](https://semver.org/). It updates to the latest release within your current major version and skips releases that include breaking changes. For example, if you are on `1.0.1` and both `1.1.0` and `2.0.0` are available, Abstrax updates to `1.1.0` and not `2.0.0`.

Update to a specific version (validated against GitHub before downloading):

```bash
sudo abstrax self update 1.2.0
```

Upgrade across a major version when you are ready for breaking changes:

```bash
sudo abstrax self update --allow-breaking
```

Preview what would happen without making changes:

```bash
abstrax self update --dry-run
```

Show download URLs and the install path:

```bash
sudo abstrax self update --verbose
```

### How version selection works

| Situation | Behaviour |
|---|---|
| No version given | Updates to the newest release with the same major version |
| Version argument given | Installs that exact version if it exists on GitHub |
| `--allow-breaking` | Installs the absolute latest release, including major version bumps |
| Already on the latest compatible release | Reports that you are up to date |
| A newer major release exists | Completes the compatible update (if one is available), then prints a notice about the major release |

If you are already on the latest compatible release but a newer major version is available, Abstrax tells you and suggests running `abstrax self update --allow-breaking` when you are ready.

### What the command does

When an update is needed, `self update` performs the same core steps as the manual release archive method:

1. Queries the GitHub releases API for published versions
2. Resolves the target version using the rules above
3. Downloads `abstrax_<version>_linux_<arch>.tar.gz` from GitHub
4. Downloads `abstrax_<version>_checksums.txt`
5. Verifies the archive SHA-256 hash against the checksums file
6. Extracts the `abstrax` binary from the archive
7. Replaces the executable that is currently running (not necessarily `/usr/local/bin/abstrax`)

Abstrax replaces the path returned by the running process, so it works whether you installed to `/usr/local/bin/abstrax`, `/usr/bin/abstrax` from a package, or another location on your `PATH`.

### Example output

After a successful update:

```text
Updated from 1.0.1 to 1.1.0.
```

When already up to date:

```text
You are already on the latest compatible release (1.1.0).
```

When a major release is also available:

```text
WARNING: A newer major release (2.0.0) is available with breaking changes. Run `abstrax self update --allow-breaking` when you are ready to upgrade.
```

JSON output includes structured fields such as `current_version`, `target_version`, `updated`, `already_up_to_date`, and `breaking_update_available`:

```bash
sudo abstrax self update --json
```

See [Self](/docs/commands/self) for full command reference.

## Option 2: Update from a release archive

This is the manual, fully transparent method. Every step is visible, so you can see exactly what is downloaded and verify it before it runs. Use this if you prefer not to use the built-in updater, or if you need to update a binary on a machine where the installed copy of Abstrax is too old to include `self update`.

Releases are published as compressed archives on the [GitHub releases page](https://github.com/useabstrax/abstrax/releases). GoReleaser builds `tar.gz` archives for `linux/amd64` and `linux/arm64`, named `abstrax_<version>_linux_<arch>.tar.gz` (for example `abstrax_1.0.0_linux_amd64.tar.gz`). Each release tag uses a `v` prefix (for example `v1.0.0`), but the version in the filename omits it.

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

To update to the latest release without looking up the version number:

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

Each release publishes a SHA-256 checksums file named `abstrax_<version>_checksums.txt`. The commands above download it and verify the archive before extracting. If the checksum check fails, do not install the binary.

If you installed to a different path, replace `/usr/local/bin/abstrax` with the path where your binary lives. To find it:

```bash
which abstrax
```

Then confirm the new version:

```bash
abstrax version
```

## Option 3: Update using the convenience script

The [install script](https://useabstrax.com/install.sh) performs the same basic steps as the manual release archive method. It always installs the **latest** release to `/usr/local/bin/abstrax`. It does not support semver-aware updates or targeting a specific version.

If you originally installed with the script, you can run it again to pick up the latest release:

```bash
wget https://useabstrax.com/install.sh
less install.sh
sudo bash install.sh
```

Or in a single command:

```bash
wget -qO- https://useabstrax.com/install.sh | sudo bash
```

The script detects architecture, downloads the archive and checksums file, verifies the SHA-256 hash, extracts the binary, and installs it. If you prefer every step to be visible on the command line, use [Option 2](#option-2-update-from-a-release-archive) instead.

## Option 4: Update a source build

If you installed by building from source, pull the latest code and rebuild:

```bash
cd abstrax/cli
git pull
go build -o abstrax ./cmd/abstrax
sudo mv abstrax /usr/local/bin/abstrax
```

See [Building from source](/docs/contributing/building-from-source) for full detail on release builds, cross-compilation, and packaging.

## Option 5: Update a package installation

If you installed Abstrax from a `.deb` or `.rpm` package, update using your package manager when a newer package is available, or use `abstrax self update` which replaces the binary at its installed path (typically `/usr/bin/abstrax`).

## Verify the update

After any update method, confirm the binary runs and reports the expected version:

```bash
abstrax version
abstrax doctor
```

`doctor` works without root and is a good smoke test that the updated binary is functioning.

## Next steps

- [Installation](/docs/getting-started/) - install Abstrax for the first time.
- [Uninstalling](/docs/getting-started/uninstalling) - remove Abstrax from your server.
- [Self command reference](/docs/commands/self) - flags and behaviour for `abstrax self update`.
