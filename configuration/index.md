# Configuration

Abstrax is controlled primarily through commands and flags. All server-wide configuration lives under `/etc/abstrax/` as JSON files.

This section covers:

- [Config file](/docs/configuration/config-file) - general settings, MySQL connection config, project state, and directories Abstrax uses.
- [Environment variables](/docs/configuration/environment-variables) - what Abstrax reads from the environment.
- [Permissions](/docs/configuration/permissions) - which commands need root and why.

## Summary

| Item | Path | Notes |
|---|---|---|
| General settings | `/etc/abstrax/config.json` | Managed by `config` commands, mode 0640 |
| MySQL connection config | `/etc/abstrax/mysql.json` | Written by `mysql install`, `mysql reset-root-password`, or `mysql config set`, mode 0600 |
| Project state | `/etc/abstrax/projects/<name>.json` | One JSON file per project, mode 0640 |
| Config directory | `/etc/abstrax` | Created with mode 0750 |
| Runtime state | `/var/lib/abstrax` | Plugin records and caches, mode 0750 |
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
| `--allow-blocked-plugin` | Allow execution of registry-blocked plugins (repeatable) |

These are described in [Environment variables](/docs/configuration/environment-variables) and the [command reference](/docs/reference/index).
