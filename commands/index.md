# Commands

- [Users](/docs/commands/users)
- [SSH keys](/docs/commands/ssh-keys)
- [Packages](/docs/commands/packages)
- [System](/docs/commands/system)
- [Cron](/docs/commands/cron)
- [Daemons](/docs/commands/daemons)
- [Projects](/docs/commands/projects)
- [Certificates](/docs/commands/certificates)

Abstrax commands are grouped by area. Every command follows the same pattern:

```text
abstrax <group> <action> [arguments] [flags]
```

This page introduces the global flags and the top-level commands, then links to detailed documentation for each command group.

## Global flags

These flags are available on every command:

| Flag | Description |
|---|---|
| `--json` | Output machine-readable JSON instead of text |
| `--dry-run` | Show the commands that would run without making changes |
| `--yes` | Skip confirmation prompts for destructive commands |
| `--quiet` | Reduce output |
| `--verbose` | Increase output, including the underlying commands being run |
| `--no-color` | Disable coloured output |

Every command also supports `--help` (alias `-h`). The root command additionally supports `--version` (alias `-v`).

## Top-level commands

| Command | Description | Root required |
|---|---|---|
| `abstrax version` | Print version, commit, and build date | No |
| `abstrax doctor` | Inspect the system and report platform capabilities | No |
| `abstrax log` | View the Abstrax log file | No |
| `abstrax agent` | Placeholder for the future hosted agent (not implemented) | No |
| `abstrax help` | Show help for any command | No |
| `abstrax completion` | Generate a shell autocompletion script | No |

The `help` and `completion` commands are provided automatically by the command framework. `abstrax completion <shell>` prints an autocompletion script for bash, zsh, fish, or PowerShell.

### version

```bash
abstrax version
abstrax version --json
```

### doctor

`abstrax doctor` reads `/etc/os-release`, detects the package manager, service manager, and firewall backend, checks whether it is running as root, and reports which managed tools are installed.

```bash
abstrax doctor
abstrax doctor --json
```

It reports:

```text
OS, version, architecture, kernel version
Package manager (apt, dnf, yum, apk, pacman)
Service manager (systemd, sysvinit)
Firewall backend (ufw, firewalld, iptables, none)
Whether running as root
Whether the platform is fully supported
Available tools: nginx, apache2, certbot, mysql, mariadb, supervisor, redis, memcached, ufw, curl, git
```

### log

`abstrax log` reads the Abstrax log file at `/var/log/abstrax/abstrax.log` using `tail`. If the file does not exist, it says so.

```bash
abstrax log
abstrax log --lines=100
abstrax log --follow
```

| Flag | Default | Description |
|---|---|---|
| `--lines` | `50` | Number of lines to show |
| `--follow`, `-f` | `false` | Follow the log output |

### agent

The `agent` command and its subcommands (`connect`, `status`, `run`, `update`) are placeholders. They print a message stating that agent mode is not yet implemented and make no changes. See [Supported platforms](/docs/reference/supported-platforms#future-agent).

## Command groups

| Group | Purpose | Documentation |
|---|---|---|
| `user` | Manage Linux users and groups | [Users](/docs/commands/users) |
| `ssh-key` | Manage SSH authorised keys for users | [SSH keys](/docs/commands/ssh-keys) |
| `ssh` | Manage SSH server configuration | [SSH keys](/docs/commands/ssh-keys#ssh-server-configuration) |
| `package` | Manage apt packages | [Packages](/docs/commands/packages) |
| `service` | Manage systemd services | [System](/docs/commands/system#service-commands) |
| `cron` | Manage scheduled cron jobs | [Cron](/docs/commands/cron) |
| `daemon` | Manage background processes with Supervisor | [Daemons](/docs/commands/daemons) |
| `project` | Manage nginx-backed web projects | [Projects](/docs/commands/projects) |
| `web` | Test, reload, and restart the web server | [Projects](/docs/commands/projects#web-server-commands) |
| `ssl` | Manage Let's Encrypt certificates with Certbot | [Certificates](/docs/commands/certificates) |
| `mysql` | Manage MySQL/MariaDB databases and users | [System](/docs/commands/system#mysql-commands) |
| `cache` | Manage Redis and Memcached | [System](/docs/commands/system#cache-commands) |
| `firewall` | Manage the UFW firewall | [System](/docs/commands/system#firewall-commands) |
| `server` | Show server status and resource usage | [System](/docs/commands/system#server-status-commands) |

For a single page listing every command and flag, see the [command reference](/docs/reference/index).

## A note on permissions

Many commands change system state and require root. Run them with `sudo`. Read-only commands such as `doctor`, `version`, `server status`, and the various `list`, `info`, and `status` actions generally do not require root, though some need root to read protected system files. Each command page notes the requirement. See [Permissions](/docs/configuration/permissions) for the full picture.
