# MySQL

The `mysql` group manages MySQL or MariaDB databases, users, and grants. Abstrax connects using a saved connection configuration stored at `/etc/abstrax/mysql.json`.

```text
abstrax mysql <action> [arguments] [flags]
abstrax mysql config <action> [flags]
abstrax mysql database <action> [arguments] [flags]
abstrax mysql user <action> [arguments] [flags]
```

## Permissions

`config set`, `install`, and `reset-root-password` require root. Other MySQL commands do not enforce a root check in code; they rely on the saved connection config to authenticate to the database. Note that reading the config file at `/etc/abstrax/mysql.json` (mode 0600, root only) typically requires root.

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

Install runs `apt install mysql-server`, enables and starts the service, applies secure defaults (removes anonymous users, drops the test database, restricts remote root), and sets the root password for both `root@localhost` (socket) and `root@127.0.0.1` (TCP). This replaces the default `auth_socket` plugin on Debian/Ubuntu so root can log in with a password from database apps and SSH tunnels.

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
abstrax mysql user list
abstrax mysql user info <name>
```

`user add` flags:

| Flag | Default | Description |
|---|---|---|
| `--host` | `localhost` | Host the user may connect from |
| `--password` | `false` | Prompt for a password instead of generating one |
| `--grant-db` | | Grant access to this database |
| `--privileges` | | Specific privileges to grant |
| `--preset` | `app` | Privilege preset (`readonly`, `app`, `admin`) |

If `--password` is omitted, a secure random password is generated and displayed once. Abstrax does not store application user passwords. Use `--password` to enter your own password at a prompt.

`user remove` asks for confirmation unless `--yes` is given. It accepts `--host` (default `localhost`).

```bash
abstrax mysql user add appuser --grant-db=myapp_db --preset=app
abstrax mysql user add appuser --password --grant-db=myapp_db --preset=app
abstrax mysql user info appuser
abstrax mysql user remove appuser
```

## Grants

```bash
abstrax mysql grant <user> <database> [flags]
abstrax mysql revoke <user> <database>
```

`grant` flags:

| Flag | Default | Description |
|---|---|---|
| `--privileges` | | Specific privileges to grant |
| `--preset` | `app` | Privilege preset (`readonly`, `app`, `admin`) |

`revoke` asks for confirmation unless `--yes` is given.

## Privilege presets

| Preset | Privileges |
|---|---|
| `readonly` | `SELECT` |
| `app` | `SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, INDEX, DROP` |
| `admin` | `ALL PRIVILEGES` |

```bash
abstrax mysql grant appuser myapp_db --preset=app
abstrax mysql revoke appuser myapp_db
```
