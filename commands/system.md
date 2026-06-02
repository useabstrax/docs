# System commands

This page documents several command groups that manage core system services:

- [`service`](#service-commands) – systemd services
- [`mysql`](#mysql-commands) – MySQL/MariaDB databases and users
- [`cache`](#cache-commands) – Redis and Memcached
- [`firewall`](#firewall-commands) – the UFW firewall
- [`server`](#server-status-commands) – server status and resource usage

---

## Service commands

The `service` group manages systemd services.

```text
abstrax service <action> <name>
```

### Permissions

`start`, `stop`, `restart`, `reload`, `enable`, and `disable` require root. `status` does not require root.

### Service names

A service name may contain letters, digits, and the characters `_ @ : . -`, up to 64 characters.

### Lifecycle commands

```bash
sudo abstrax service start <name>
sudo abstrax service stop <name>
sudo abstrax service restart <name>
sudo abstrax service reload <name>
sudo abstrax service enable <name>
sudo abstrax service disable <name>
```

- `enable` configures the service to start at boot.
- `disable` stops it starting at boot.

Note: `stop` and `restart` interrupt the service. Restarting a service that handles live traffic (such as a web server or database) causes a brief outage. Prefer `reload` where the service supports it.

### `service status`

```bash
abstrax service status nginx
```

```text
  Service:      nginx
  Active:       active
  Sub:          running
  Enabled:      enabled
  PID:          12345
  Description:  A high performance web server
```

---

## MySQL commands

The `mysql` group manages MySQL or MariaDB databases, users, and grants. Abstrax connects using a saved connection configuration stored at `/etc/abstrax/mysql.toml`.

```text
abstrax mysql <action> [arguments] [flags]
abstrax mysql config <action> [flags]
abstrax mysql database <action> [arguments] [flags]
abstrax mysql user <action> [arguments] [flags]
```

### Permissions

`config set` and `install` require root. Other MySQL commands do not enforce a root check in code; they rely on the saved connection config to authenticate to the database. Note that reading the config file at `/etc/abstrax/mysql.toml` (mode 0600, root only) typically requires root.

### Connection configuration

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

### Test the connection

```bash
abstrax mysql test
```

Runs `SELECT 1` against the configured connection and reports success or failure.

### Install MySQL/MariaDB

```bash
sudo abstrax mysql install
sudo abstrax mysql install --secure
```

| Flag | Description |
|---|---|
| `--version` | Version to install |
| `--secure` | Run `mysql_secure_installation` after installing |

### Databases

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

### Users

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

### Grants

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

### Privilege presets

| Preset | Privileges |
|---|---|
| `readonly` | `SELECT` |
| `app` | `SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, INDEX, DROP` |
| `admin` | `ALL PRIVILEGES` |

```bash
abstrax mysql grant appuser myapp_db --preset=app
abstrax mysql revoke appuser myapp_db
```

---

## Cache commands

The `cache` group installs and manages Redis or Memcached.

```text
abstrax cache <action> [driver] [flags]
```

The driver is `redis` or `memcached`.

### Permissions

`install`, `remove`, `start`, `stop`, and `restart` require root. `status` and `config` do not require root.

### `cache install`

```bash
sudo abstrax cache install <driver> [flags]
```

| Flag | Default | Description |
|---|---|---|
| `--version` | | Package version |
| `--port` | `0` | Port to listen on (0 uses the driver default) |
| `--bind` | `127.0.0.1` | IP address to bind to |
| `--memory` | | Memory limit |
| `--enable` | `true` | Enable the service at boot |
| `--start` | `true` | Start the service after install |

If you set `--bind=0.0.0.0` for Redis, Abstrax warns that this exposes Redis on all interfaces and asks for confirmation (unless `--yes` is given).

```bash
sudo abstrax cache install redis
sudo abstrax cache install memcached
sudo abstrax cache install redis --bind=127.0.0.1 --memory=256mb
```

### `cache remove`

```bash
sudo abstrax cache remove <driver> [flags]
```

| Flag | Description |
|---|---|
| `--purge` | Purge configuration files |

### Lifecycle commands

```bash
sudo abstrax cache start <driver>
sudo abstrax cache stop <driver>
sudo abstrax cache restart <driver>
```

### `cache status`

Show status for all drivers, or a single driver. Does not require root.

```bash
abstrax cache status
abstrax cache status redis
```

```text
DRIVER     RUNNING   ENABLED
redis      yes       yes
memcached  no        no
```

### `cache config`

Show configuration information for a driver. Does not require root. The current output reports the location of the driver's main configuration file (for example `/etc/redis/redis.conf` or `/etc/memcached.conf`); structured config management is not yet available.

```bash
abstrax cache config redis
```

---

## Firewall commands

The `firewall` group manages the UFW firewall.

```text
abstrax firewall <action> [arguments] [flags]
abstrax firewall rule <action> [arguments]
```

### Permissions

`enable`, `disable`, `allow`, `deny`, `allow-ip`, `deny-ip`, and `rule remove` require root. `status` and `rule list` do not require root.

### `firewall status`

Show whether the firewall is active and list its rules. Does not require root.

```bash
abstrax firewall status
```

```text
  Firewall:    ufw
  Status:      active

  Rules:
    [1] 22/tcp               ALLOW
    [2] 443/tcp              ALLOW
```

### `firewall enable`

Enable the firewall. Abstrax warns that enabling the firewall without opening SSH may lock you out, and asks for confirmation unless `--yes` is given.

```bash
sudo abstrax firewall enable --allow-ssh
sudo abstrax firewall enable --allow-ssh --ssh-port=2222
```

| Flag | Default | Description |
|---|---|---|
| `--allow-ssh` | `false` | Open the SSH port before enabling |
| `--ssh-port` | `22` | SSH port to allow |

Always use `--allow-ssh` (and `--ssh-port` if you changed it) unless you have another way into the server.

### `firewall disable`

```bash
sudo abstrax firewall disable
```

Asks for confirmation unless `--yes` is given.

### Allowing and denying ports

```bash
sudo abstrax firewall allow <port> [flags]
sudo abstrax firewall deny <port> [flags]
```

`allow` flags:

| Flag | Description |
|---|---|
| `--protocol` | Protocol (`tcp` or `udp`) |
| `--from` | Allow only from this IP or CIDR |
| `--comment` | Rule comment |

`deny` supports `--protocol`.

The behaviour differs between `allow` and `deny`:

- `firewall allow <port>` validates the port as a plain number in the range 1–65535. A value such as `443/tcp` is rejected with `port must be a number`. To restrict the protocol, use the `--protocol` flag.
- `firewall deny <port>` does not validate the port, so a value is passed through to UFW as given.

```bash
sudo abstrax firewall allow 80
sudo abstrax firewall allow 443 --protocol=tcp
sudo abstrax firewall deny 23
```

### Allowing and denying IPs

```bash
sudo abstrax firewall allow-ip <ip-or-cidr> [flags]
sudo abstrax firewall deny-ip <ip-or-cidr>
```

The IP or CIDR is validated. `allow-ip` supports `--to` (destination IP) and `--port` (specific port).

```bash
sudo abstrax firewall allow-ip 192.168.1.0/24
sudo abstrax firewall deny-ip 10.0.0.5
```

### Managing rules

```bash
abstrax firewall rule list
sudo abstrax firewall rule remove <id>
```

`rule list` shows the numbered rules. `rule remove` removes a rule by its number.

```text
ID   PORT/IP     PROTOCOL   ACTION
1    22          tcp        ALLOW
2    443         tcp        ALLOW
```

---

## Server status commands

The `server` group reports server status and resource usage. These commands are read-only and do not require root.

```text
abstrax server <action> [flags]
```

### `server status`

Show a comprehensive overview.

```bash
abstrax server status
```

The output includes hostname, uptime, load average, CPU cores, memory usage, swap usage (if present), OS information, kernel version, private IP addresses, and disk usage.

```text
  Hostname:            web01
  Uptime:              up 3 days, 4 hours
  Load average:        0.15  0.10  0.05

  CPU:                 4 cores
  Memory:              7976 MB total / 2104 MB used (26.4%)
  Swap:                2048 MB total / 0 MB used (0.0%)

  OS:    Ubuntu 22.04.3 LTS
  Kernel: 5.15.0-91-generic

  Private IPs:
    10.0.0.5

  Disk usage:
    /                    12.3 GB / 50.0 GB (25%)
```

### Other server commands

```bash
abstrax server cpu
abstrax server memory
abstrax server disk
abstrax server disk --path=/var/www
abstrax server load
abstrax server services
abstrax server services --failed
```

| Command | Description |
|---|---|
| `server cpu` | Number of CPU cores |
| `server memory` | Total, used, free memory and usage percentage |
| `server disk` | Disk usage per mount point; `--path` limits to one path |
| `server load` | Load average (1, 5, 15 minutes) |
| `server services` | List services; `--failed` shows only failed services |

## Related

- [Daemons](/commands/daemons.html) – for Supervisor-managed processes
- [Packages](/commands/packages.html)
- [Configuration](/configuration/index.html)
- [Security](/reference/security.html)
