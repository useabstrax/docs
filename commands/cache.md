# Cache

The `cache` group installs and manages Redis or Memcached.

```text
abstrax cache <action> [driver] [flags]
```

The driver is `redis` or `memcached`.

## Permissions

`install`, `remove`, `start`, `stop`, and `restart` require root. `status` and `config` do not require root.

## `cache install`

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

## `cache remove`

```bash
sudo abstrax cache remove <driver> [flags]
```

| Flag | Description |
|---|---|
| `--purge` | Purge configuration files |

## Lifecycle commands

```bash
sudo abstrax cache start <driver>
sudo abstrax cache stop <driver>
sudo abstrax cache restart <driver>
```

## `cache status`

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

## `cache config`

Show configuration information for a driver. Does not require root. The current output reports the location of the driver's main configuration file (for example `/etc/redis/redis.conf` or `/etc/memcached.conf`); structured config management is not yet available.

```bash
abstrax cache config redis
```
