# Config file

Abstrax does not use a general configuration file to control its behaviour. There is one configuration file, used only by the MySQL commands, plus a set of directories where Abstrax stores state and logs.

## MySQL connection config

The MySQL commands need to know how to connect to the database server. That connection information is saved in:

```text
/etc/abstrax/mysql.toml
```

You create it with:

```bash
sudo abstrax mysql config set --host=127.0.0.1 --user=root --password
```

The file is written with mode `0600`, so only its owner (root) can read it. It uses a simple key/value format:

```toml
# Abstrax MySQL config - root readable only
host = "127.0.0.1"
port = 3306
user = "root"
password = "..."
socket = ""
```

Notes:

- The password is only written if you provided one with `--password` (which prompts securely). It is never passed on the command line.
- `abstrax mysql config show` displays the host, port, user, and socket, but not the password.
- If no config file exists, the MySQL commands fall back to the defaults `host=127.0.0.1`, `port=3306`, `user=root` with no password.

See the [MySQL commands](/docs/commands/system#mysql-commands) for usage.

## Directories Abstrax uses

| Directory | Purpose | Permissions |
|---|---|---|
| `/etc/abstrax` | Configuration (currently `mysql.toml`) | 0750 |
| `/var/lib/abstrax` | Runtime state | 0750 |
| `/var/lib/abstrax/projects` | Project state, one JSON file per project | |
| `/var/log/abstrax` | Logs | 0750 |

When you install the binary by hand, the directories are created as needed by the commands that use them (for example `mysql config set` creates `/etc/abstrax`).

## Project state files

Each project you create with `abstrax project add` is recorded as a JSON file:

```text
/var/lib/abstrax/projects/<name>.json
```

This file stores the project's name, path, domains, web server, runtime, SSL status, virtual host path, owner, and timestamps. The `project list` and `project info` commands read these files. You should not normally edit them by hand; use the `project modify` command instead.

## Other system files Abstrax writes

Abstrax also writes to standard system locations when you use the relevant commands:

| File or directory | Written by |
|---|---|
| `/etc/ssh/sshd_config.d/99-abstrax.conf` | `ssh config` commands |
| `/etc/cron.d/abstrax-<id>` | `cron add` / `cron modify` |
| `/etc/supervisor/conf.d/abstrax-<name>.conf` | `daemon add` / `daemon modify` |
| `/etc/nginx/sites-available/abstrax-<name>` | `project add` |
| `/etc/nginx/sites-enabled/abstrax-<name>` | `project enable` |

These are not Abstrax configuration files as such; they are the configuration of the underlying tools that Abstrax manages on your behalf.

## Related

- [Environment variables](/docs/configuration/environment-variables)
- [Permissions](/docs/configuration/permissions)
- [Security](/docs/reference/security)
