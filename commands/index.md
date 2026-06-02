# Commands

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

The `agent` command and its subcommands (`connect`, `status`, `run`, `update`) are placeholders. They print a message stating that agent mode is not yet implemented and make no changes. See [Supported platforms](reference/supported-platforms.html#future-agent).

## Command groups

| Group | Purpose | Documentation |
|---|---|---|
| `user` | Manage Linux users and groups | [users.md](commands/users.html) |
| `ssh-key` | Manage SSH authorised keys for users | [ssh-keys.md](commands/ssh-keys.html) |
| `ssh` | Manage SSH server configuration | [ssh-keys.md](commands/ssh-keys.html#ssh-server-configuration) |
| `package` | Manage apt packages | [packages.md](commands/packages.html) |
| `service` | Manage systemd services | [system.md](commands/system.html#service-commands) |
| `cron` | Manage scheduled cron jobs | [cron.md](commands/cron.html) |
| `daemon` | Manage background processes with Supervisor | [daemons.md](commands/daemons.html) |
| `project` | Manage nginx-backed web projects | [projects.md](commands/projects.html) |
| `web` | Test, reload, and restart the web server | [projects.md](commands/projects.html#web-server-commands) |
| `ssl` | Manage Let's Encrypt certificates with Certbot | [certificates.md](commands/certificates.html) |
| `mysql` | Manage MySQL/MariaDB databases and users | [system.md](commands/system.html#mysql-commands) |
| `cache` | Manage Redis and Memcached | [system.md](commands/system.html#cache-commands) |
| `firewall` | Manage the UFW firewall | [system.md](commands/system.html#firewall-commands) |
| `server` | Show server status and resource usage | [system.md](commands/system.html#server-status-commands) |

For a single page listing every command and flag, see the [command reference](reference/command-reference.html).

## A note on permissions

Many commands change system state and require root. Run them with `sudo`. Read-only commands such as `doctor`, `version`, `server status`, and the various `list`, `info`, and `status` actions generally do not require root, though some need root to read protected system files. Each command page notes the requirement. See [Permissions](configuration/permissions.html) for the full picture.
