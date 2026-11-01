# MySQL

The `mysql` group manages MySQL or MariaDB databases, users, and grants. Abstrax connects using a saved connection configuration stored at `/etc/abstrax/mysql.json`.

```text
abstrax mysql <action> [arguments] [flags]
abstrax mysql config <action> [flags]
abstrax mysql database <action> [arguments] [flags]
abstrax mysql user <action> [arguments] [flags]
abstrax mysql remote <action> [flags]
```

## Permissions

`config set`, `install`, `reset-root-password`, `remote enable`, and `remote disable` require root. `remote status` does not. Other MySQL commands do not enforce a root check in code; they rely on the saved connection config to authenticate to the database. Note that reading the config file at `/etc/abstrax/mysql.json` (mode 0600, root only) typically requires root.

## Connection configuration

```bash
sudo abstrax mysql config set --host=127.0.0.1 --user=root --password
abstrax mysql config show
```

`config set` flags:

| Flag | Default | Description |
|---|---|---|
| `--host` | `127.0.0.1` | MySQL host |
| `--port` | `3306` | MySQL port |
| `--user` | `root` | MySQL user |
| `--password` | `false` | Prompt for the password securely |
| `--socket` | | MySQL socket path |

The password is entered at a prompt, never as a command-line argument. The config file is written with mode `0600` (readable only by its owner). `config show` does not display the password.

## Test the connection

```bash
abstrax mysql test
```

Runs `SELECT 1` against the configured connection and reports success or failure.

## Install MySQL/MariaDB

```bash
sudo abstrax mysql install
sudo abstrax mysql install --root-password='YourPassword'
```

| Flag | Description |
|---|---|
| `--version` | Version to install |
| `--root-password` | Root password (a secure random password is generated if omitted) |

Install behaviour depends on the platform family:

- **Debian/Ubuntu family:** installs `mysql-server` via `apt`, enables/starts the `mysql` service, and sets root password auth with `caching_sha2_password` (replacing Debian's default `auth_socket` where present).
- **RHEL-compatible family:** installs MariaDB as the MySQL-compatible database server (`mariadb-server` via `dnf`), enables/starts `mariadb`, and sets root password auth with MariaDB-compatible `IDENTIFIED BY` SQL. The CLI command remains `mysql` for compatibility; this is not Oracle MySQL.

In both cases Abstrax applies secure defaults (removes anonymous users, drops the test database, restricts remote root) and sets the root password for both `root@localhost` (socket) and `root@127.0.0.1` (TCP).

On RHEL-family systems, `--version` is ignored and the stock AppStream MariaDB package is installed.


The root password is displayed once after install and is saved to `/etc/abstrax/mysql.json` so Abstrax commands such as `mysql database add` work immediately without a separate `config set` step.

For automation, you can set `ABSTRAX_MYSQL_ROOT_PASSWORD` instead of `--root-password`. Note that a password passed on the command line may be visible in process listings.

If MySQL is already configured with a password, install will not overwrite it. Use `reset-root-password` if you have lost the root password.

Removing the package with `package remove mysql-server` does **not** reset MySQL. The database files in `/var/lib/mysql` are kept, so reinstalling and running `mysql install` again will reuse the existing data and root password. For a completely fresh install, purge the package and remove the data directory:

```bash
sudo abstrax package remove mysql-server --purge
sudo rm -rf /var/lib/mysql
sudo abstrax mysql install
```

Or keep the existing data and reset the root password instead:

```bash
sudo abstrax mysql reset-root-password --yes
```

## Reset root password

```bash
sudo abstrax mysql reset-root-password
sudo abstrax mysql reset-root-password --root-password='NewPassword' --yes
```

| Flag | Description |
|---|---|
| `--root-password` | New root password (generated if omitted) |
| `--yes` | Skip confirmation prompt |

Requires root on the host. Does not require knowing the current MySQL root password. The command stops MySQL, starts it briefly in recovery mode, sets a new root password for both `root@localhost` and `root@127.0.0.1` (replacing `auth_socket` if present), and restarts the service normally.

The new password is displayed once and is saved to `/etc/abstrax/mysql.json` so Abstrax commands continue to work without running `config set` again.

For automation, `ABSTRAX_MYSQL_ROOT_PASSWORD` can be used instead of `--root-password`.

## Remote access

```bash
sudo abstrax mysql remote enable
sudo abstrax mysql remote enable --bind=192.0.2.10
sudo abstrax mysql remote disable
abstrax mysql remote status
```

| Flag | Default | Description |
|---|---|---|
| `--bind` | `0.0.0.0` | IP address the database server should listen on (`enable` only) |
| `--yes` | | Skip the confirmation prompt (`enable` only) |
| `--dry-run` | | Show the change without writing a file or restarting |

`remote enable` and `remote disable` require root. `remote status` does not.

### Drop-in

Enable writes an Abstrax-managed file named `zz-abstrax-remote.cnf`. The `zz-` prefix makes it load last, so this `bind-address` overrides an earlier setting in the same include directory. The directory comes from the platform's database config paths, not a fixed distro path:

- Ubuntu (MySQL): a directory ending in `mysql.conf.d`, usually `/etc/mysql/mysql.conf.d`
- Debian (MariaDB): `mariadb.conf.d`, usually `/etc/mysql/mariadb.conf.d`
- RHEL-family (MariaDB): `/etc/my.cnf.d`

The file looks like this:

```ini
[mysqld]
bind-address = 0.0.0.0
```

On MySQL, Abstrax also sets `mysqlx-bind-address` to the same address. MariaDB does not get that line. `--bind` chooses a specific interface address instead of `0.0.0.0`. The value must be an IP address, with no spaces, quotes, or line breaks.

### Confirmation

Before enabling, Abstrax warns that the server will accept connections beyond loopback and asks you to continue. `--yes` skips the prompt. This is the same kind of warning `cache install` shows before binding Redis to `0.0.0.0`.

Abstrax does not change the firewall. If remote clients should connect, allow TCP port 3306 yourself (for example with `firewall allow 3306`).

### Restart and rollback

After writing or removing the drop-in, Abstrax restarts the database service and waits until it accepts connections. If the service does not come back, enable deletes the drop-in and restarts again. Disable puts the file back and restarts again. The database is not left down because of this command.

`--dry-run` writes nothing and restarts nothing.

### Disable when the setting is not Abstrax's

`remote disable` only removes `zz-abstrax-remote.cnf`. If the running server is bound to something other than loopback and that value does not come from Abstrax's file, the command says so and does not claim that remote access was turned off.

### Status

`remote status` reads the effective value from the running server (`SHOW VARIABLES LIKE 'bind_address'`), not only whether the drop-in exists. `enabled` is true only when that address is something other than loopback (`127.0.0.1` or `::1`). If an administrator set `bind-address` in another file, status still reports the value the server is using.

```json
{ "enabled": true, "bind_address": "0.0.0.0", "managed_file": "/etc/mysql/mysql.conf.d/zz-abstrax-remote.cnf", "port": 3306 }
```

If the database is stopped, the command does not fail. It reads the drop-in (when one exists) and reports the live bind address as `unknown`.

## Databases

```bash
abstrax mysql database add <name> [flags]
abstrax mysql database remove <name>
abstrax mysql database list
```

`database add` flags:

| Flag | Default | Description |
|---|---|---|
| `--charset` | `utf8mb4` | Character set |
| `--collation` | `utf8mb4_unicode_ci` | Collation |
| `--if-not-exists` | `false` | Do not error if the database already exists |

`database remove` is destructive and asks for confirmation unless `--yes` is given. Database names may contain letters, digits, and underscores, up to 64 characters.

```bash
abstrax mysql database add myapp_db --charset=utf8mb4 --if-not-exists
abstrax mysql database remove myapp_db
```

## Users

```bash
abstrax mysql user add <name> [flags]
abstrax mysql user remove <name> [flags]
abstrax mysql user list [flags]
abstrax mysql user info <name> [flags]
```

`user add` flags:

| Flag | Default | Description |
|---|---|---|
| `--host` | `localhost` | Host the user may connect from. When `localhost`, Abstrax creates both `'user'@'localhost'` and `'user'@'127.0.0.1'` with the same password and grants. |
| `--password` | `false` | Prompt for a password instead of generating one |
| `--grant-db` | | Grant access to this database |
| `--privileges` | | Specific privileges to grant |
| `--preset` | `app` | Privilege preset (`readonly`, `app`, `admin`) |

When `--host` is `localhost` (the default), Laravel and other apps that connect via TCP to `127.0.0.1` still work because MySQL treats that as a separate account from `'user'@'localhost'` (socket).

If `--password` is omitted, a secure random password is generated and displayed once. Abstrax does not store application user passwords. Use `--password` to enter your own password at a prompt.

`user remove` asks for confirmation unless `--yes` is given. It accepts `--host` (default `localhost`).

Database names passed to `--grant-db` are checked the same way as `mysql grant`: letters, digits, and underscores only, up to 64 characters. The host must match a safe character set (letters, digits, and `.` `%` `_` `:` `/` `-`) and is escaped in SQL.

```bash
abstrax mysql user add appuser --grant-db=myapp_db --preset=app
abstrax mysql user add appuser --password --grant-db=myapp_db --preset=app
abstrax mysql user info appuser
abstrax mysql user remove appuser
```

### `user list`

Without flags, the list is each account's name and host.

```bash
abstrax mysql user list
abstrax mysql user list --grants
abstrax mysql user list --grants --exclude-system
```

| Flag | Description |
|---|---|
| `--grants` | Include each user's raw grants and the parsed privileges for each database |
| `--exclude-system` | Hide built-in accounts |

With `--grants`, the text table adds a `DATABASES` column. `--json` adds `grants` (the raw `SHOW GRANTS` lines), `databases` (name, privileges, and a matching preset when one applies), and `global_privileges`. Without `--grants`, those fields are omitted.

### `user info`

Show grants for every host the user exists on, including a specific IP. JSON output is a list, one entry per host.

```bash
abstrax mysql user info appuser
abstrax mysql user info appuser --host=10.0.4.12
```

| Flag | Description |
|---|---|
| `--host` | Limit the result to one host |

## Grants

```bash
abstrax mysql grant <user> <database> [flags]
abstrax mysql revoke <user> <database>
```

`grant` flags:

| Flag | Default | Description |
|---|---|---|
| `--privileges` | | Specific privileges to grant (overrides `--preset`) |
| `--preset` | `app` when `--privileges` is omitted | Privilege preset (`readonly`, `app`, `admin`) |
| `--host` | local app hosts | Limit the grant to one host, including a specific IP |

When `--host` is omitted, Abstrax grants on the local accounts (`'user'@'localhost'` and `'user'@'127.0.0.1'`). If only one local account exists, Abstrax creates the missing one by copying the authentication hash from the existing account. Pass `--host` to grant a different account, such as `'user'@'10.0.4.12'`.

Privileges are checked against an allowlist and the `GRANT` statement is built from that list. Column-level privileges, such as `SELECT(id)`, are rejected. Hosts must use letters, digits, and `.` `%` `_` `:` `/` `-`, and are escaped in SQL.

`revoke` accepts `--host` with the same semantics and asks for confirmation unless `--yes` is given.

## Privilege presets

| Preset | Privileges |
|---|---|
| `readonly` | `SELECT` |
| `app` | `SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, INDEX, DROP, REFERENCES` |
| `admin` | `ALL PRIVILEGES` |

```bash
abstrax mysql grant appuser myapp_db --preset=app
abstrax mysql revoke appuser myapp_db
```
