# Packages

The `package` command group manages system packages through the platform package manager (`apt` on Debian-family systems, `dnf` on RHEL-family systems).

```text
abstrax package <action> [arguments] [flags]
```

## Permissions

Commands that change installed packages (`install`, `remove`, `update`, `upgrade`, `auto-updates enable`, `auto-updates disable`) require root. The read-only commands `search`, `info`, `list`, `upgrades`, and `auto-updates status` do not require root.

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

Update the package lists (equivalent to `apt update` or `dnf makecache`).

```bash
sudo abstrax package update
```

## `package upgrade`

Upgrade installed packages. Use `--security-only` to install security updates only.

```bash
sudo abstrax package upgrade
sudo abstrax package upgrade --security-only
```

| Flag | Description |
|---|---|
| `--security-only` | Install security updates only |

Without `--security-only`, Abstrax runs a full upgrade (`apt-get upgrade` on Debian and Ubuntu, `dnf upgrade` on RHEL-family systems).

With `--security-only`:

- Debian and Ubuntu: installs only the packages `package upgrades --security-only` would list. If none are pending, the command succeeds without installing anything.
- RHEL-family: runs `dnf upgrade -y --security`. When the package manager is yum, it runs `yum update -y --security`.

## `package upgrades`

List pending upgrades. Does not require root.

```bash
abstrax package upgrades
abstrax package upgrades --security-only
abstrax package upgrades --json
```

| Flag | Description |
|---|---|
| `--security-only` | Show security updates only |

Each row includes the package name, the installed version, the new version, and whether the update is a security update.

On Debian and Ubuntu, Abstrax parses `apt list --upgradable`. A package is a security update when the candidate version or the origin contains `-security` (the security pocket). On RHEL-family systems, pending upgrades come from `dnf check-update` (or `yum check-update`). A package is a security update when `dnf updateinfo list --security` lists it (`yum updateinfo list security` when yum is the package manager). The installed version is read with `rpm`.

```text
NAME                  INSTALLED              NEW                    SECURITY
openssl               3.0.2-0ubuntu1.15      3.0.2-0ubuntu1.18      yes
nginx                 1.18.0-6ubuntu14.3     1.18.0-6ubuntu14.4     no
```

`--json` returns that list as `data`:

```json
[
  {
    "name": "openssl",
    "installed_version": "3.0.2-0ubuntu1.15",
    "new_version": "3.0.2-0ubuntu1.18",
    "security": true
  }
]
```

## `package auto-updates`

Manage automatic security updates. Updates are limited to security origins, and the server is never rebooted automatically. Use [`server reboot-required`](/docs/commands/server#server-reboot-required) when a reboot is needed.

Registered as `package.auto-updates.status`, `package.auto-updates.enable`, and `package.auto-updates.disable`.

### `package auto-updates status`

Show whether automatic security updates are enabled. Changes nothing and does not require root.

```bash
abstrax package auto-updates status
abstrax package auto-updates status --json
```

```text
  Backend:       unattended-upgrades
  Installed:     yes
  Enabled:       yes
  Managed:       yes
  Last run:      2026-11-03T06:12:00Z
  Last result:   success
```

| Field | Description |
|---|---|
| `backend` | `unattended-upgrades` (Debian/Ubuntu) or `dnf-automatic` (RHEL-family) |
| `installed` | Whether the backend package is installed |
| `enabled` | Whether automatic upgrades are turned on |
| `managed` | Whether Abstrax owns the configuration |
| `config_path` | Path to the Abstrax-managed config file when present |
| `last_run` | RFC3339 UTC timestamp of the last run, or `null` |
| `last_result` | `success`, `failed`, `unknown`, or `never_run` |

### `package auto-updates enable`

Enable automatic security updates. Requires root. Installs the backend when it is missing and writes Abstrax-owned configuration (distro files are left alone).

```bash
sudo abstrax package auto-updates enable
```

**Debian and Ubuntu:** installs `unattended-upgrades` when missing and writes `/etc/apt/apt.conf.d/52abstrax-auto-updates`. That file sets `APT::Periodic::Unattended-Upgrade "1"`, replaces the distro origin lists with security origins only, and sets `Automatic-Reboot "false"`. Status reads the effective setting from `apt-config`. The last run is the mtime of `/var/lib/apt/periodic/unattended-upgrades-stamp`; the result comes from the last outcome line in `/var/log/unattended-upgrades/unattended-upgrades.log`.

**RHEL-family:** installs `dnf-automatic`, writes `/etc/abstrax/dnf-automatic.conf` (`upgrade_type = security`, `apply_updates = yes`, `reboot = never`), and adds a systemd drop-in `/etc/systemd/system/dnf-automatic-install.service.d/50-abstrax.conf` that runs `dnf-automatic` with that file. Then it runs `systemctl enable --now dnf-automatic-install.timer`. The last run is the mtime of `/var/lib/systemd/timers/stamp-dnf-automatic-install.timer`. The result comes from `systemctl show dnf-automatic-install.service` and is only known for runs since boot; an earlier run reports `unknown`.

### `package auto-updates disable`

Disable automatic security updates. Requires root. The backend package stays installed.

```bash
sudo abstrax package auto-updates disable
```

**Debian and Ubuntu:** rewrites the Abstrax apt config with `APT::Periodic::Unattended-Upgrade "0"`. Distro files are not edited.

**RHEL-family:** stops and disables the timer, removes both Abstrax files, and reloads systemd. Leaves `/etc/dnf/automatic.conf` untouched.

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

- These commands use the detected package manager (`apt` or `dnf`). Run `abstrax doctor` to confirm which backend is active.
- Long-running operations such as `upgrade` may take time. Use `--verbose` to see the underlying command being run.

## Related

- [Installing packages](/docs/guides/installing-packages)
- [Cache (Redis, Memcached)](/docs/commands/cache)
- [Troubleshooting](/docs/reference/troubleshooting)
