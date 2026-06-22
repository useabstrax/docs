# Config file

Abstrax stores all configuration in `/etc/abstrax/` as JSON files. Most commands read and write these files automatically; you normally manage general settings with the [`config`](/docs/commands/config) command rather than editing files by hand.

## General settings

Server-wide Abstrax settings are stored in:

```text
/etc/abstrax/config.json
```

View effective settings (including built-in defaults):

```bash
abstrax config show
```

Example file after customising PHP extensions:

```json
{
  "php": {
    "extensions": ["mysql", "xml", "curl", "mbstring", "zip", "bcmath", "gd", "intl", "redis", "readline"]
  }
}
```

Extension values are apt package suffixes. When PHP is installed for a project, Abstrax expands them to versioned packages (for example `mysql` becomes `php8.5-mysql` for PHP 8.5).

If the file does not exist, Abstrax uses built-in defaults. See the [config command](/docs/commands/config) for `set`, `add`, `remove`, and `reset`.

### Plugin settings

Plugin registry configuration can be stored in `config.json`:

```json
{
  "plugins": {
    "registry_url": "https://plugins.useabstrax.com/api/v1",
    "allow_blocked": ["legacy-plugin"]
  }
}
```

| Key | Description |
|---|---|
| `registry_url` | Base URL for the plugin registry (default: `https://plugins.useabstrax.com/api/v1`) |
| `allow_blocked` | Plugin names permitted to run despite blocked registry status |

These keys are read from the config file but are not managed by `abstrax config set`. Edit `/etc/abstrax/config.json` directly, or set `ABSTRAX_PLUGIN_REGISTRY` to override the registry URL for a single command. The `allow_blocked` list is merged with any `--allow-blocked-plugin` flags.

See [Plugins](/docs/plugins/) for how plugins work and how to build them.

### Project settings

User isolated projects can use custom paths outside a user's home only when the path is inside an approved shared root:

```json
{
  "projects": {
    "approved_roots": ["/srv/sites", "/srv/www"]
  }
}
```

Configure with:

```bash
sudo abstrax config set projects.approved_roots /srv/sites /srv/www
```

## MySQL connection config

The MySQL commands need to know how to connect to the database server. That connection information is saved in:

```text
/etc/abstrax/mysql.json
```

You create or update it with:

```bash
sudo abstrax mysql config set --host=127.0.0.1 --user=root --password
```

`mysql install` and `mysql reset-root-password` write this file automatically. Use `config set` only when you need to change connection settings manually (for example, a non-default host or user).

The file is written with mode `0600`, so only its owner (root) can read it. Example contents:

```json
{
  "host": "127.0.0.1",
  "port": 3306,
  "user": "root",
  "password": "..."
}
```

Notes:

- The password is only written if you provided one with `--password` (which prompts securely). It is never passed on the command line.
- `abstrax mysql config show` displays the host, port, user, and socket, but not the password.
- If no config file exists, the MySQL commands fall back to the defaults `host=127.0.0.1`, `port=3306`, `user=root` with no password.
- Upgrading from an older Abstrax version that used `mysql.toml` migrates the file to `mysql.json` automatically on first use.

See the [MySQL](/docs/commands/mysql) commands for usage.

## Project state files

Each project you create with `abstrax project add` is recorded as a JSON file:

```text
/etc/abstrax/projects/<name>.json
```

This file stores the project's name, path, domains, web server, runtime, runtime version (PHP, Node.js, or Ruby as applicable), public directory, proxy port, SSL status, virtual host path, owner, and timestamps. The `project list` and `project info` commands read these files. You should not normally edit them by hand; use the `project modify` command instead.

Upgrading from an older Abstrax version that stored projects under `/var/lib/abstrax/projects/` migrates them to `/etc/abstrax/projects/` automatically on first use.

## Directories Abstrax uses

| Directory | Purpose | Permissions |
|---|---|---|
| `/etc/abstrax` | Configuration (`config.json`, `mysql.json`, `projects/`) | 0750 |
| `/etc/abstrax/projects` | Project state, one JSON file per project | 0640 per file |
| `/var/lib/abstrax` | Runtime state (plugin records and caches) | 0750 |
| `/var/lib/abstrax/plugins` | Plugin installation records and caches | |
| `/usr/local/lib/abstrax/plugins` | Installed plugin binaries (system) | |
| `/var/log/abstrax` | Logs | 0750 |

When you install the binary by hand, the directories are created as needed by the commands that use them (for example `mysql config set` creates `/etc/abstrax`).

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
