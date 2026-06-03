# Configuration

Abstrax is controlled almost entirely through commands and flags. It does not read a general settings file to change its behaviour. There is one specific configuration file, used only by the MySQL commands, and a small set of standard directories that Abstrax uses for state and logs.

This section covers:

- [Config file](/docs/configuration/config-file) – the MySQL connection config and the directories Abstrax uses.
- [Environment variables](/docs/configuration/environment-variables) – what Abstrax reads from the environment.
- [Permissions](/docs/configuration/permissions) – which commands need root and why.

## Summary

| Item | Path | Notes |
|---|---|---|
| MySQL connection config | `/etc/abstrax/mysql.toml` | Written by `mysql config set`, mode 0600 |
| Config directory | `/etc/abstrax` | Created with mode 0750 |
| Project state | `/var/lib/abstrax/projects/<name>.json` | One JSON file per project |
| State directory | `/var/lib/abstrax` | Created with mode 0750 |
| Log file | `/var/log/abstrax/abstrax.log` | Read by `abstrax log` |
| Log directory | `/var/log/abstrax` | Created with mode 0750 |

There is no global config file that changes default flags, output format, or behaviour. Everything else is provided per command through flags. See the [command reference](/docs/reference/index) for the full list.

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
