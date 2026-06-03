# Packages

- [Users](/docs/commands/users)
- [SSH keys](/docs/commands/ssh-keys)
- [Packages](/docs/commands/packages)
- [System](/docs/commands/system)
- [Cron](/docs/commands/cron)
- [Daemons](/docs/commands/daemons)
- [Projects](/docs/commands/projects)
- [Certificates](/docs/commands/certificates)

The `package` command group manages system packages through `apt`. It is intended for Debian and Ubuntu based systems.

```text
abstrax package <action> [arguments] [flags]
```

## Permissions

Commands that change installed packages (`install`, `remove`, `update`, `upgrade`) require root. The read-only commands `search`, `info`, and `list` do not require root.

## Package name validation

Package names are validated before any change. A name may contain letters, digits, and the characters `_ . + -`, up to 128 characters.

## `package install`

Install a package, optionally pinning a version.

```bash
sudo abstrax package install <name> [flags]
```

| Flag | Description |
|---|---|
| `--version` | Specific package version to install |

```bash
sudo abstrax package install nginx
sudo abstrax package install nginx --version=1.24.0-1ubuntu1
```

## `package remove`

Remove a package. Use `--purge` to also remove its configuration files.

```bash
sudo abstrax package remove <name> [flags]
```

| Flag | Description |
|---|---|
| `--purge` | Purge configuration files as well as the package |

```bash
sudo abstrax package remove nginx
sudo abstrax package remove nginx --purge
```

## `package update`

Update the package lists (equivalent to `apt update`).

```bash
sudo abstrax package update
```

## `package upgrade`

Upgrade installed packages. Use `--security-only` to apply security updates only.

```bash
sudo abstrax package upgrade
sudo abstrax package upgrade --security-only
```

| Flag | Description |
|---|---|
| `--security-only` | Apply security updates only |

## `package search`

Search for packages. Does not require root.

```bash
abstrax package search <query>
```

### Example output

```text
NAME       DESCRIPTION
nginx      small, powerful, scalable web/proxy server
nginx-core nginx web/proxy server (standard version)
```

## `package info`

Show details for a package. Does not require root.

```bash
abstrax package info <name>
```

```text
  Name:          nginx
  Version:       1.24.0-1ubuntu1
  Architecture:  amd64
  Status:        installed
  Description:   small, powerful, scalable web/proxy server
```

## `package list`

List installed packages. Does not require root.

```bash
abstrax package list
```

```text
NAME       VERSION              ARCHITECTURE
nginx      1.24.0-1ubuntu1      amd64
curl       8.5.0-2ubuntu10      amd64
```

## Notes

- These commands use `apt`. On systems without `apt`, they will not work. Run `abstrax doctor` to confirm the detected package manager.
- Long-running operations such as `upgrade` may take time. Use `--verbose` to see the underlying command being run.

## Related

- [Installing packages](/docs/guides/installing-packages)
- [Cache (Redis, Memcached)](/docs/commands/system#cache-commands)
- [Troubleshooting](/docs/reference/troubleshooting)
