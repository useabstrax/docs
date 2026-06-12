# MySQL

The `mysql` group manages MySQL or MariaDB databases, users, and grants. Abstrax connects using a saved connection configuration stored at `/etc/abstrax/mysql.toml`.

```text
abstrax mysql <action> [arguments] [flags]
abstrax mysql config <action> [flags]
abstrax mysql database <action> [arguments] [flags]
abstrax mysql user <action> [arguments] [flags]
```

## Permissions

`config set` and `install` require root. Other MySQL commands do not enforce a root check in code; they rely on the saved connection config to authenticate to the database. Note that reading the config file at `/etc/abstrax/mysql.toml` (mode 0600, root only) typically requires root.

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
sudo abstrax mysql install --secure
```

| Flag | Description |
|---|---|
| `--version` | Version to install |
| `--secure` | Run `mysql_secure_installation` after installing |

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
| `--password` | `false` | Prompt for the password securely |
| `--grant-db` | | Grant access to this database |
| `--privileges` | | Specific privileges to grant |
| `--preset` | `app` | Privilege preset (`readonly`, `app`, `admin`) |

`user remove` asks for confirmation unless `--yes` is given. It accepts `--host` (default `localhost`).

```bash
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
