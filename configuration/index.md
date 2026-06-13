# Configuration

Abstrax is controlled primarily through commands and flags. Server-wide settings live in `/etc/abstrax/config.json` and are managed with the [`config`](/docs/commands/config) command. MySQL connection settings use a separate file.

This section covers:

- [Config file](/docs/configuration/config-file) - general settings, MySQL connection config, and directories Abstrax uses.
- [Environment variables](/docs/configuration/environment-variables) - what Abstrax reads from the environment.
- [Permissions](/docs/configuration/permissions) - which commands need root and why.

## Summary

| Item | Path | Notes |
|---|---|---|
| General settings | `/etc/abstrax/config.json` | Managed by `config` commands, mode 0640 |
| MySQL connection config | `/etc/abstrax/mysql.toml` | Written by `mysql config set`, mode 0600 |
| Config directory | `/etc/abstrax` | Created with mode 0750 |
| Project state | `/var/lib/abstrax/projects/<name>.json` | One JSON file per project |
| State directory | `/var/lib/abstrax` | Created with mode 0750 |
| Log file | `/var/log/abstrax/abstrax.log` | Read by `abstrax log` |
| Log directory | `/var/log/abstrax` | Created with mode 0750 |

Global CLI behaviour (output format, dry-run, confirmations) is controlled per command through flags, not the config file. See the [command reference](/docs/reference/index) for the full list.

## Behaviour controlled by flags

Instead of a config file, Abstrax uses global flags to control output and safety:

| Flag | Effect |
|---|---|
| `--json` | Output structured JSON |
| `--dry-run` | Preview changes without applying them |
| `--yes` | Skip confirmation prompts |
| `--quiet` | Reduce output |
| `--verbose` | Show the underlying commands being run |
| `--no-color` | Disable coloured output |

These are described in [Environment variables](/docs/configuration/environment-variables) and the [command reference](/docs/reference/index).
