# Self

The `self` command group manages the Abstrax CLI itself. It is separate from `package update` and `package upgrade`, which manage system packages through `apt`.

## Permissions

`self update` requires root because it replaces the running binary, which is typically installed in a system directory such as `/usr/local/bin` or `/usr/bin`.

```bash
sudo abstrax self update
```

`--dry-run` does not require root because it only resolves the target version and reports what would happen.

## `self update`

Update the Abstrax CLI from GitHub releases.

```bash
sudo abstrax self update [version] [flags]
```

| Argument / flag | Description |
|---|---|
| `[version]` | Optional target version (without a leading `v`). Must exist on GitHub. |
| `--allow-breaking` | Upgrade to the latest release even when it includes breaking (major version) changes |

### Default behaviour

When no version is given, Abstrax uses [semantic versioning](https://semver.org/) to choose a target:

- It looks at all published, non-prerelease versions on GitHub.
- It selects the newest release that shares your current major version.
- It skips major version bumps unless you pass `--allow-breaking`.

For example, on version `1.0.1` with `1.1.0` and `2.0.0` available, the default target is `1.1.0`.

When you pass a version explicitly, Abstrax validates that the release exists on GitHub before downloading. An explicit version is installed even if it is a major version bump.

### What happens during an update

1. The current version is read from the running binary.
2. Published releases are fetched from the GitHub API.
3. The target version is resolved.
4. If no update is needed, Abstrax reports that you are up to date.
5. Otherwise it downloads the release archive and checksums file, verifies the SHA-256 hash, extracts the binary, and replaces the running executable.

Abstrax only supports `linux/amd64` and `linux/arm64`.

### Examples

Update to the newest compatible release:

```bash
sudo abstrax self update
```

Update to a specific version:

```bash
sudo abstrax self update 1.2.0
```

Upgrade to the absolute latest release, including major versions:

```bash
sudo abstrax self update --allow-breaking
```

Preview without making changes:

```bash
abstrax self update --dry-run
```

JSON output:

```bash
sudo abstrax self update --json
```

```json
{
  "status": "success",
  "action": "self.update",
  "summary": "Updated from 1.0.1 to 1.1.0.",
  "data": {
    "current_version": "1.0.1",
    "target_version": "1.1.0",
    "latest_version": "2.0.0",
    "breaking_version": "2.0.0",
    "updated": true,
    "already_up_to_date": false,
    "breaking_update_available": true,
    "message": "Updated from 1.0.1 to 1.1.0.",
    "notice": "A newer major release (2.0.0) is available with breaking changes. Run `abstrax self update --allow-breaking` when you are ready to upgrade."
  }
}
```

When already up to date:

```bash
sudo abstrax self update
```

```text
You are already on the latest compatible release (1.1.0).
WARNING: A newer major release (2.0.0) is available with breaking changes. Run `abstrax self update --allow-breaking` when you are ready to upgrade.
```

### Errors

Common errors:

| Error | Cause |
|---|---|
| `this command requires root privileges` | Run with `sudo` |
| `version X.Y.Z does not exist on GitHub` | The requested version was not found among published releases |
| `self-update is only supported on Linux` | The command was run on a non-Linux system |
| `checksum mismatch` | The downloaded archive failed verification; the update was aborted |
| `unsupported architecture` | The server architecture is not `amd64` or `arm64` |

## Related

- [Updating](/docs/getting-started/updating) - full update guide including manual archive steps.
- [Installation](/docs/getting-started/) - first-time installation.
